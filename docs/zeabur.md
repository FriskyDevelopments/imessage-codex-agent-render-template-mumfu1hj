# Deploy on Zeabur

Runs the same stack as `render.yaml` on a Zeabur dedicated server: one Node
service, one PostgreSQL, one volume. No code changes versus Render.

| Render | Zeabur |
| --- | --- |
| `preDeployCommand: npm run db:migrate` | `npm run start:zeabur` runs `node dist/db/migrate.js` before `node dist/server.js` (migrations are idempotent) |
| Disk `codex-data` at `/var/data` | Volume at `/var/data` |
| `imessage-agent-db` (Postgres 18) | Zeabur PostgreSQL service in the same project |
| `APP_ENCRYPTION_KEY: generateValue` | Set by hand once (below) |
| `DEPLOYMENT_ID` derived from `RENDER_SERVICE_ID` | Set by hand once to a fixed UUID (below) |

Build/start commands come from `zbpack.json`.

## Steps

1. **Project** on the target server → **Add Service → Marketplace → PostgreSQL**.
2. **Add Service → GitHub →** this repo, branch `zeabur`.
3. **Volumes:** mount id `data` at `/var/data`.
4. **Variables** on the agent service:

   | Key | Value |
   | --- | --- |
   | `NODE_ENV` | `production` |
   | `DATABASE_URL` | the PostgreSQL service's connection string (reference it, e.g. `${POSTGRES_CONNECTION_STRING}`) |
   | `CODEX_HOME` | `/var/data/codex` |
   | `AGENT_WORKSPACE_ROOT` | `/var/data/workspaces` |
   | `CODEX_AUTH_MODE` | `chatgpt` |
   | `DEPLOYMENT_ID` | a UUID, generated once and **never changed** (`uuidgen`) |
   | `APP_ENCRYPTION_KEY` | `openssl rand -hex 32`, generated once and **never changed** — rotating it makes stored credentials unreadable |

   `PORT` is injected by Zeabur; the server binds to it.

5. **Networking:** expose the HTTP port and generate or bind a domain.
6. Deploy. `/healthz` returns 200 as soon as the process is up. `/readyz` turns
   200 after you open the domain and save the owner phone, authenticate Photon,
   and connect ChatGPT — same flow as the Render README.

Keep the service at **one instance**: Codex credentials and workspaces live on
the volume.
