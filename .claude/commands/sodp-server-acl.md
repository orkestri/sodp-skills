Generate an ACL (Access Control List) configuration for the SODP server.

SODP ACL rules control which users can read or write each state key. Rules are evaluated in order — the first matching rule wins. If no rule matches, access is allowed (permissive default).

`$ARGUMENTS` can describe the use case, e.g.:
- `game` — players can only write their own state, admins can write anything
- `collab` — authenticated users read everything, write only their own slot
- `saas` — tenant isolation, each tenant sees only their data

## What to do

1. **Ask the user** (or infer from `$ARGUMENTS`) what access control they need:
   - What are the key namespaces? (e.g. `game.*`, `user.*`, `room.*`)
   - Who should be able to read? (everyone, authenticated users, specific roles)
   - Who should be able to write? (owner only, specific roles, anyone)
   - Is there multi-tenancy? (each user/tenant has their own subtree)

2. **Generate `config/acl.json`** — create the `config/` directory if it doesn't exist.

   ACL rule format:
   ```json
   [
     {
       "pattern": "dot.separated.key.pattern",
       "read":  ["permission-expression"],
       "write": ["permission-expression"]
     }
   ]
   ```

   Pattern syntax:
   - `user.{sub}` — `{sub}` captures the JWT `sub` claim; matches `user.alice`, `user.bob`, etc.
   - `admin.*` — `*` matches one or more remaining segments
   - `public.scores` — exact literal match

   Permission expressions:
   - `"*"` — anyone (including unauthenticated)
   - `"{sub}"` — only the user whose `sub` matches the captured `{sub}` in the pattern
   - `"role:admin"` — users with `role: admin` in their JWT claims
   - `"group:editors"` — users in the `editors` group
   - `"perm:write_scores"` — users with a specific permission

3. **Common patterns to generate based on the use case:**

   **Per-user private state** (each user can only access their own subtree):
   ```json
   [
     { "pattern": "user.{sub}.*", "read": ["{sub}"], "write": ["{sub}"] },
     { "pattern": "user.{sub}",   "read": ["{sub}"], "write": ["{sub}"] }
   ]
   ```

   **Public read, authenticated write:**
   ```json
   [
     { "pattern": "public.*", "read": ["*"],    "write": ["role:editor"] },
     { "pattern": "admin.*",  "read": ["role:admin"], "write": ["role:admin"] }
   ]
   ```

   **Game rooms (players in room, admins everywhere):**
   ```json
   [
     { "pattern": "room.{sub}.*", "read": ["*"],          "write": ["{sub}", "role:admin"] },
     { "pattern": "admin.*",      "read": ["role:admin"],  "write": ["role:admin"] },
     { "pattern": "leaderboard",  "read": ["*"],           "write": ["role:gameserver"] }
   ]
   ```

   **Multi-tenant SaaS (tenant isolation):**
   ```json
   [
     { "pattern": "tenant.{sub}.*", "read": ["{sub}", "role:superadmin"], "write": ["{sub}", "role:superadmin"] }
   ]
   ```

   **Collaborative editing (read all, write own slot):**
   ```json
   [
     { "pattern": "collab.*.cursors",  "read": ["*"], "write": ["*"] },
     { "pattern": "collab.*.doc",      "read": ["*"], "write": ["role:editor"] },
     { "pattern": "collab.*",          "read": ["*"], "write": ["role:editor"] }
   ]
   ```

4. **Tell the user how to load the ACL file on the server:**

   Rust server:
   ```bash
   SODP_ACL_FILE=config/acl.json ./sodp-server 0.0.0.0:7777
   ```

   Go embedded:
   ```go
   srv := sodp.NewServer(
       sodp.WithACLFile("config/acl.json"),
   )
   ```

   Docker:
   ```bash
   docker run -p 7777:7777 \
     -e SODP_ACL_FILE=/config/acl.json \
     -v $(pwd)/config:/config \
     ghcr.io/orkestri/sodp-server:latest
   ```

5. **Remind the user** that ACL enforcement requires JWT authentication to be enabled — without a token, `sub` is empty and role/group claims are unavailable.
