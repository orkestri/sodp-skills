Embed the SODP Go server into this Go HTTP service.

The Go SODP server (`github.com/orkestri/sodp-go`) is a library — you attach it to your existing `http.ServeMux` or router. Clients connect over WebSocket; your server-side code calls `srv.Mutate()` to push state changes to all watchers.

`$ARGUMENTS` can specify: a router/framework (`chi`, `gin`, `echo`, `fiber`, `stdlib`), or options like `jwt`, `acl`, `persist`, `redis`.

## What to do

1. **Check `go.mod`** to detect the HTTP router in use (chi, gin, echo, fiber, or stdlib).

2. **Add the dependency:**
   ```bash
   go get github.com/orkestri/sodp-go
   ```

3. **Detect whether the project already has an HTTP server setup.** Find the main file or server initialization.

4. **Create or update the server setup** to mount the SODP handler:

   **stdlib / net/http:**
   ```go
   package main

   import (
       "net/http"
       "os"

       sodp "github.com/orkestri/sodp-go"
   )

   func main() {
       srv := sodp.NewServer(
           sodp.WithBackpressureLimit(1024),
           // sodp.WithJWTSecret([]byte(os.Getenv("SODP_JWT_SECRET"))),
           // sodp.WithACLFile("config/acl.json"),
           // sodp.WithPersistenceDir("/var/lib/sodp"),
       )
       defer srv.Close()

       http.HandleFunc("/sodp", srv.HandleWS)
       http.HandleFunc("/health", srv.HealthHandler().ServeHTTP)

       http.ListenAndServe(":7777", nil)
   }
   ```

   **chi:**
   ```go
   r := chi.NewRouter()
   r.Get("/sodp", srv.HandleWS)
   r.Get("/health", srv.HealthHandler().ServeHTTP)
   ```

   **gin:**
   ```go
   r := gin.Default()
   r.GET("/sodp", gin.WrapF(srv.HandleWS))
   r.GET("/health", gin.WrapH(srv.HealthHandler()))
   ```

   **echo:**
   ```go
   e := echo.New()
   e.GET("/sodp", echo.WrapHandler(http.HandlerFunc(srv.HandleWS)))
   ```

5. **Show how to push state from server-side code:**
   ```go
   // Replace full value — triggers DELTA to all watchers
   srv.Mutate("game.score", map[string]any{"value": 42, "player": "alice"})

   // Append element to an array, cap at 500
   srv.MutateAppend("game.events", GameEvent{Type: "goal"}, 500)

   // Delete a key
   srv.MutateDelete("game.player")
   ```

6. **Add JWT auth if requested** (`$ARGUMENTS` contains `jwt`):
   ```go
   srv := sodp.NewServer(
       sodp.WithJWTSecret([]byte(os.Getenv("SODP_JWT_SECRET"))),
       // or RS256:
       // sodp.WithJWTPublicKey(os.Getenv("SODP_JWT_PUBLIC_KEY")),
   )
   ```

7. **Add persistence if requested** (`$ARGUMENTS` contains `persist`):
   ```go
   srv := sodp.NewServer(
       sodp.WithPersistenceDir("/var/lib/sodp"),
   )
   ```

8. **Add Redis cluster if requested** (`$ARGUMENTS` contains `redis`):
   ```go
   import "github.com/orkestri/sodp-go/sodpredis"

   backend, err := sodpredis.New(os.Getenv("SODP_REDIS_URL"))
   if err != nil { log.Fatal(err) }

   srv := sodp.NewServer(
       sodp.WithCluster(backend),
   )
   ```

9. **Available server options summary:**
   | Option | Purpose |
   |--------|---------|
   | `WithJWTSecret([]byte)` | HS256 token validation |
   | `WithJWTPublicKey(pem)` | RS256 token validation |
   | `WithACLFile(path)` | JSON ACL rules |
   | `WithPersistenceDir(dir)` | Replay log for RESUME |
   | `WithMaxSessions(n)` | Connection limit |
   | `WithBackpressureLimit(n)` | Outbound channel capacity per session |
   | `WithRateLimit(n)` | Max writes/sec per session |
   | `WithMaxWatches(n)` | Max subscriptions per session |
   | `WithCluster(backend)` | Horizontal scaling |
   | `WithAllowedOrigins([]string)` | WebSocket origin check |

10. **Tell the user** what was added and suggest:
    - Run `/sodp-server-acl` to generate ACL rules
    - Run `/sodp-server-schema` to add schema validation
    - Run `/sodp-setup` to integrate a client in the frontend or another service
