# evolve-server

Community backend for the Evolve Revival project. Implements the subset of the 2K kando API v1.8.1.3 that the game requires: service discovery, authentication, entitlements, player storage, matchmaking, and peer coordination.

## Endpoints

| Service | Path prefix | Description |
|---------|-------------|-------------|
| Doorman | `/doorman/1/` | Service directory — tells the client where to reach each downstream service |
| SSO | `/sso/1/` | Authentication — accepts Steam ID + display name, returns a session token |
| Entitlements | `/entitlements/1/` | DLC ownership and app entitlement checks |
| Storage | `/storage/1/` | Per-player key-value data (properties, unlocks, sessions, replays) |
| Peers | `/peers/1/` | Matchmaking queue wait times and peer coordination |
| Grants | `/grants/1/` | Entitlement grants |
| Stats | `/stats/1/` | Stat group configuration (stub) |
| Telemetry | `/evolve/event` | Analytics sink (stub, always 200) |

All responses (except doorman) are wrapped in the kando RPC envelope:

```json
{
  "result": { ... },
  "header": { "code": 0, "cache": { "onlineTtl": 86400, "offlineTtl": -1 } }
}
```

Doorman returns flat JSON — the client reads `services` and `clientConfigSettings` from the top level.

## Stack

- Go + Gin
- PostgreSQL (player records, sessions, storage datasets)

## Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `8080` | HTTP listen port |
| `DATABASE_URL` | `postgres://evolve:evolve@localhost/evolve?sslmode=disable` | PostgreSQL DSN |
| `SERVER_HOST` | `localhost:8080` | Hostname written into the doorman service directory |
| `RELAY_PORT` | `47584` | UDP port for peer relay |

## Running

```bash
make run
```

Apply migrations:

```bash
DATABASE_URL=<dsn> make migrate
```

Run tests:

```bash
make test
```

## Docker

```bash
docker build -t evolve-server .
docker run -e DATABASE_URL=<dsn> -p 8080:8080 evolve-server
```

## Related

- [evolve-launcher](https://github.com/evolve-revival/evolve-launcher) — local launcher that serves the kando API directly to the game process on `127.0.0.1:443`
