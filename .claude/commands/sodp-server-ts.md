Embed the SODP TypeScript server into this Node.js application.

`@sodp/server` is a library — you instantiate `SodpServer`, configure it with callbacks (auth, ACL, schema, hydration, custom RPCs), and either call `server.listen(port)` or attach it to an existing HTTP server. All state mutations fan out as DELTA frames to every active watcher.

`$ARGUMENTS` can specify: a framework (`express`, `fastify`, `hono`, `koa`, `standalone`), or options like `jwt`, `acl`, `schema`, `persist`, `redis`.

## What to do

1. **Check `package.json`** to detect the HTTP framework in use (Express, Fastify, Hono, Koa, or standalone).

2. **Install the dependency:**
   ```bash
   npm install @sodp/server ws @msgpack/msgpack jose
   ```

3. **Detect whether the project already has an HTTP server setup.** Find the entry point or server initialization file.

4. **Create or update the server setup.**

   **Standalone (no existing HTTP server):**
   ```typescript
   import { SodpServer } from "@sodp/server";

   const server = new SodpServer({
     jwtSecret: process.env.SODP_JWT_SECRET,
   });

   await server.listen(7777);
   server.handleSignals(); // graceful SIGTERM / SIGINT shutdown
   ```

   **Express:**
   ```typescript
   import express from "express";
   import { createServer } from "http";
   import { SodpServer } from "@sodp/server";

   const app = express();
   const http = createServer(app);

   const sodp = new SodpServer({ jwtSecret: process.env.SODP_JWT_SECRET });
   sodp.attach(http, { path: "/sodp" });
   sodp.handleSignals();

   http.listen(3000, () => console.log("listening on :3000"));
   ```

   **Fastify:**
   ```typescript
   import Fastify from "fastify";
   import { SodpServer } from "@sodp/server";

   const app = Fastify();
   const sodp = new SodpServer({ jwtSecret: process.env.SODP_JWT_SECRET });

   app.addHook("onReady", async () => {
     sodp.attach(app.server, { path: "/sodp" });
   });
   sodp.handleSignals();
   await app.listen({ port: 3000 });
   ```

   **Hono (Node.js adapter):**
   ```typescript
   import { serve } from "@hono/node-server";
   import { Hono } from "hono";
   import { SodpServer } from "@sodp/server";

   const app = new Hono();
   const sodp = new SodpServer({ jwtSecret: process.env.SODP_JWT_SECRET });

   const server = serve({ fetch: app.fetch, port: 3000 }, (info) => {
     sodp.attach(server, { path: "/sodp" });
   });
   sodp.handleSignals();
   ```

5. **Show how to push state from server-side code:**
   ```typescript
   // Replace full value — triggers DELTA to all watchers
   server.set("game.score", { value: 42, player: "alice" });

   // Merge fields only
   server.patch("game.player", { health: 80 });

   // Set a nested field
   server.setIn("game.player", "/position/x", 5);

   // Append to an array, cap at 500
   server.append("game.events", { type: "goal", ts: Date.now() }, 500);

   // Delete a key
   server.delete("game.player");
   ```

6. **Add JWT auth if requested** (`$ARGUMENTS` contains `jwt`):
   ```typescript
   new SodpServer({
     // HS256
     jwtSecret: process.env.SODP_JWT_SECRET,
     // RS256
     // jwtPublicKey: process.env.SODP_JWT_PUBLIC_KEY,
     // Built-in IdP presets — no manual claim mapping
     // jwtPreset: "keycloak", // or auth0 | okta | cognito | generic
   })
   ```

7. **Add ACL if requested** (`$ARGUMENTS` contains `acl`):
   ```typescript
   new SodpServer({ aclFile: "config/acl.json" })
   ```
   Run `/sodp-server-acl` to generate the ACL file.

8. **Add schema validation if requested** (`$ARGUMENTS` contains `schema`):
   ```typescript
   new SodpServer({ schemaFile: "config/schema.json" }) // ERROR 422 on violations
   ```
   Run `/sodp-server-schema` to generate the schema file.

9. **Add persistence if requested** (`$ARGUMENTS` contains `persist`):
   ```typescript
   new SodpServer({ persistDir: "/var/lib/sodp" }) // 100ms-debounced JSON snapshots
   ```

10. **Add Redis cluster if requested** (`$ARGUMENTS` contains `redis`):
    ```bash
    npm install ioredis
    ```
    ```typescript
    new SodpServer({ redisUrl: process.env.SODP_REDIS_URL }) // e.g. "redis://127.0.0.1/"
    ```

11. **Add hydration if the project has a database** (seed state when a key is first watched):
    ```typescript
    new SodpServer({
      hydrate: async (key) => {
        return await db.getState(key); // return null if not found → initialized: false
      },
    })
    ```

12. **Add custom RPC if requested:**
    ```typescript
    new SodpServer({
      onCall: async (method, args, claims) => {
        if (method === "room.join") return { success: true, data: await joinRoom(args.roomId, claims.sub) };
        return { success: false, data: `unknown method: ${method}` };
      },
    })
    ```

13. **Available options summary:**
    | Option | Purpose |
    |--------|---------|
    | `jwtSecret` | HS256 shared secret |
    | `jwtPublicKey` | RS256 public key PEM |
    | `jwtPreset` | `keycloak` / `auth0` / `okta` / `cognito` / `generic` |
    | `claimMappings` | Override preset claim paths |
    | `authenticate` | Custom async auth callback |
    | `aclFile` | JSON ACL rules path |
    | `authorize` | Custom async ACL callback |
    | `schemaFile` | JSON schema path — ERROR 422 on violations |
    | `persistDir` | Snapshot directory |
    | `redisUrl` | Redis URL for horizontal scaling |
    | `hydrate` | Seed state from DB on first watch |
    | `onCall` | Custom RPC handler |
    | `maxPayload` | Max incoming frame bytes (default 1 MiB) |
    | `backpressureLimit` | Outbound queue depth per session (default 1024) |
    | `rateLimitWrites` | Max writes/sec per session |
    | `rateLimitWatches` | Max new watches/sec per session |
    | `healthPort` | `GET /health` HTTP port |
    | `metricsPort` | `GET /metrics` Prometheus port |

14. **Environment variables** (all optional):
    | Variable | Purpose |
    |----------|---------|
    | `SODP_JWT_SECRET` | HS256 secret |
    | `SODP_JWT_PUBLIC_KEY` | RS256 public key PEM (inline) |
    | `SODP_JWT_PUBLIC_KEY_FILE` | RS256 public key file path |
    | `SODP_ACL_FILE` | ACL JSON file path |
    | `SODP_REDIS_URL` | Redis connection URL |
    | `SODP_HEALTH_PORT` | Health check HTTP port |
    | `SODP_METRICS_PORT` | Prometheus metrics HTTP port |
    | `SODP_MAX_PAYLOAD` | Max incoming frame bytes |
    | `SODP_BACKPRESSURE_LIMIT` | Outbound queue depth per session |
    | `SODP_RATE_WRITES_PER_SEC` | Write rate limit per session |
    | `SODP_RATE_WATCHES_PER_SEC` | Watch rate limit per session |

15. **Tell the user** what was added and suggest:
    - Run `/sodp-server-acl` to generate ACL rules
    - Run `/sodp-server-schema` to add schema validation
    - Run `/sodp-setup` to integrate a client in the frontend or another service
