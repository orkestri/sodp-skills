Generate a SODP schema validation configuration for state keys.

SODP validates state values server-side before applying them. Invalid writes receive `ERROR 422` — the state is not modified. Unknown keys (not declared in the schema) are allowed by default (permissive).

`$ARGUMENTS` can describe the domain, e.g.:
- `game` — player health, score, position
- `collab` — document content, cursor positions
- Or a specific key: `game.player`

## What to do

1. **Ask the user** (or infer from `$ARGUMENTS`) what state keys exist and what shape their values have.

2. **Generate `config/schema.json`** — create the `config/` directory if it doesn't exist.

   Schema file format — a JSON object mapping state keys to type nodes:

   **Simple types** (just a string):
   ```json
   { "metrics.uptime": "Int" }
   ```

   **Full node** (with nullability and nested fields):
   ```json
   {
     "game.player": {
       "type": "Object",
       "nullable": false,
       "fields": {
         "name":   { "type": "String",  "nullable": false },
         "health": { "type": "Int",     "nullable": false },
         "score":  { "type": "Float",   "nullable": false },
         "active": { "type": "Bool",    "nullable": false },
         "position": {
           "type": "Object",
           "nullable": false,
           "fields": {
             "x": { "type": "Float", "nullable": false },
             "y": { "type": "Float", "nullable": false }
           }
         }
       }
     }
   }
   ```

   Available types:
   | Type | Description |
   |------|-------------|
   | `"String"` | JSON string |
   | `"Int"` | JSON integer |
   | `"Float"` | JSON number (Int is also accepted for Float fields) |
   | `"Bool"` | JSON boolean |
   | `"Object"` | JSON object — can have `fields` for nested validation |
   | `"Array"` | JSON array |
   | `"Any"` | Any JSON value — disables type checking for this field |

   Rules:
   - `"nullable": true` allows `null` for that field
   - Extra fields in the value not declared in the schema are allowed (permissive)
   - Keys not declared in the schema — all values pass

3. **Generate a realistic schema** based on the described use case:

   **Game state example:**
   ```json
   {
     "game.player": {
       "type": "Object",
       "fields": {
         "name":     { "type": "String" },
         "health":   { "type": "Int" },
         "score":    { "type": "Int" },
         "position": {
           "type": "Object",
           "fields": {
             "x": { "type": "Float" },
             "y": { "type": "Float" }
           }
         }
       }
     },
     "game.settings": {
       "type": "Object",
       "fields": {
         "maxPlayers": { "type": "Int" },
         "roundTime":  { "type": "Int" }
       }
     },
     "game.leaderboard": "Array"
   }
   ```

   **Collaborative editor example:**
   ```json
   {
     "collab.doc": {
       "type": "Object",
       "fields": {
         "title":   { "type": "String" },
         "content": { "type": "String" },
         "version": { "type": "Int" }
       }
     },
     "collab.cursors": "Any"
   }
   ```

4. **Tell the user how to load the schema on the server:**

   Rust server:
   ```bash
   # With persistence
   ./sodp-server 0.0.0.0:7777 /var/lib/sodp/log config/schema.json

   # Schema only (no persistence) — pass empty string for log dir
   ./sodp-server 0.0.0.0:7777 "" config/schema.json
   ```

   Go embedded:
   ```go
   // Schema validation is applied automatically when ACL + validation hook is set.
   // Use WithAuthorizeKey for custom validation, or rely on client-side type safety.
   ```

   Docker:
   ```bash
   docker run -p 7777:7777 \
     -v $(pwd)/config:/config \
     ghcr.io/orkestri/sodp-server:latest \
     0.0.0.0:7777 /data /config/schema.json
   ```

5. **Remind the user** that:
   - Schema validation runs before the state is applied — rejected writes never reach watchers
   - The error response is `ERROR 422` with a descriptive message
   - Adding schema to existing keys does not affect already-stored values — only new writes are validated
