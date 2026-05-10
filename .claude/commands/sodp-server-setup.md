Set up a SODP server for this project.

SODP has two production-ready server implementations:
- **Rust** — reference implementation, highest throughput, lowest latency
- **Go** — idiomatic Go, can run standalone or be embedded as a library

`$ARGUMENTS` can specify a preference: `rust`, `go`, `docker`, `binary`, or `embedded`.

## What to do

1. **Ask the user** (or infer from `$ARGUMENTS`):
   - Which server: Rust or Go?
   - How to run it: Docker, pre-built binary, or build from source?
   - Do they need persistence (replay missed deltas on reconnect)?
   - Do they need JWT authentication?

2. **Generate the appropriate setup:**

---

### Docker (fastest to get running — works for both Rust and Go)

```bash
docker run -p 7777:7777 ghcr.io/orkestri/sodp-server:latest
```

With persistence:
```bash
docker run -p 7777:7777 \
  -v /var/lib/sodp:/data \
  ghcr.io/orkestri/sodp-server:latest \
  0.0.0.0:7777 /data
```

With JWT + ACL:
```bash
docker run -p 7777:7777 \
  -e SODP_JWT_SECRET=your-secret \
  -e SODP_ACL_FILE=/config/acl.json \
  -v $(pwd)/config:/config \
  ghcr.io/orkestri/sodp-server:latest
```

---

### Rust server — from source

```bash
git clone https://github.com/orkestri/SODP
cd SODP
cargo build --release

# Ephemeral (no persistence)
./target/release/sodp-server 0.0.0.0:7777

# With persistence
./target/release/sodp-server 0.0.0.0:7777 /var/lib/sodp/log

# With persistence + schema validation
./target/release/sodp-server 0.0.0.0:7777 /var/lib/sodp/log config/schema.json
```

Environment variables:
| Variable | Purpose |
|----------|---------|
| `SODP_JWT_SECRET` | HS256 shared secret |
| `SODP_JWT_PUBLIC_KEY_FILE` | RS256 public key PEM path |
| `SODP_ACL_FILE` | ACL rules JSON path |
| `SODP_HEALTH_PORT` | HTTP health check port |
| `SODP_METRICS_PORT` | Prometheus metrics port |
| `SODP_RATE_WRITES_PER_SEC` | Write rate limit per session |
| `SODP_RATE_WATCHES_PER_SEC` | Watch rate limit per session |
| `SODP_BACKPRESSURE_LIMIT` | Outbound channel capacity per session (default 1024) |
| `SODP_REDIS_URL` | Redis URL for horizontal scaling |

---

### Go server — standalone binary

```bash
git clone https://github.com/orkestri/SODP
cd SODP/sodp-go
go build -o sodp-server ./cmd/sodp-server

./sodp-server -addr :7777
./sodp-server -addr :7777 -persist /var/lib/sodp
./sodp-server -addr :7777 -jwt-secret $JWT_SECRET -acl config/acl.json
```

---

3. **If the project has a `docker-compose.yml`**, add a `sodp` service to it:

```yaml
services:
  sodp:
    image: ghcr.io/orkestri/sodp-server:latest
    ports:
      - "7777:7777"
    environment:
      SODP_JWT_SECRET: ${SODP_JWT_SECRET}
    volumes:
      - sodp-data:/data
    command: ["0.0.0.0:7777", "/data"]

volumes:
  sodp-data:
```

4. **If the project has a `.env` or `.env.example`**, add:
```
SODP_URL=ws://localhost:7777
SODP_JWT_SECRET=change-me
```

5. **Tell the user** what was set up and suggest next steps:
   - Run `/sodp-server-acl` to protect state keys per user/role
   - Run `/sodp-server-schema` to add schema validation
   - Run `/sodp-setup` to integrate a client into this codebase
