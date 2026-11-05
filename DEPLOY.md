# Deploy Mao

## Local

1. Install Node 22 (Node 20 also works for local development).
2. In this directory run `npm ci`, then `npm run dev`.
3. Open `http://localhost:3000`. Open a second browser profile to join a room.
4. `/dev` uses `local-mao` unless `DEV_PASSPHRASE` is set. Change this for deployment.
5. Optional: set environment variables in your shell. `.env.example` is documentation, not automatically loaded. With Node 22 you can use `node --env-file=.env server.js`.

## Northflank Sandbox — primary target

Northflank's published Sandbox plan currently describes always-on services, with 2 services, 2 jobs and 1 database, within a free resource allowance. Check the current dashboard pricing before provisioning storage. Free compute does not automatically imply free persistent volumes.
Source: https://northflank.com/pricing

1. Push the contents of this directory to a GitHub repository (Dockerfile at repository root).
2. Sign in at `https://app.northflank.com`, create a team if prompted, and choose the Sandbox plan.
3. Create a project named `mao`, choosing the region nearest your players.
4. In the project choose **Create new → Service → Combined service** (build and deploy).
5. Connect GitHub, authorise access to your repository, select it and the main branch.
6. Choose **Dockerfile** as build type. Set Dockerfile path to `/Dockerfile` and build context to `/`.
7. Keep **one instance**. Do not enable autoscaling: rooms are intentionally held in a single process.
8. Choose a Sandbox-eligible compute size within your plan allowance.
9. Under environment variables set `PORT=3000`, `DATA_DIR=/app/storage`, and a long, random `DEV_PASSPHRASE`.
10. Under networking add a public HTTP port, container port `3000`, with HTTPS enabled. WebSockets use this same port; no second socket port is needed.
11. Add an HTTP readiness/liveness check on `/healthz`, port 3000.
12. For restart-persistent rule edits, add a persistent volume mounted at `/app/storage`. Ensure the volume is writable by UID 1000 (the container's `node` user). If Sandbox does not include persistent volumes in your account, a volume may incur cost; do not claim disk persistence without one. Room state never persists.
13. Create/deploy the service and wait for Healthy. Open the generated public hostname.
14. Set `PUBLIC_ORIGIN` to that full HTTPS origin and redeploy if desired. All client connections use the page origin, not this variable, and need no rebuild.
15. Verify `/healthz`, create a private table, copy the invitation and join from another browser/device. Set a unique developer passphrase before sharing.
16. Deploying a new image or restarting deliberately ends in-memory rooms. Export the library from `/dev` as an additional backup.

Dashboard labels can change. No cloud deployment was performed by the build agent; these are deployment instructions, not a claim of a provisioned service.

## Render fallback

1. Push to GitHub.
2. In Render choose **New → Blueprint**, connect the repository and select `render.yaml`.
3. Set `PUBLIC_ORIGIN` to the eventual `https://…onrender.com` origin (or temporarily blank).
4. Apply the blueprint and open the generated service URL after the health check succeeds.
5. Retrieve/configure the generated `DEV_PASSPHRASE` in the environment panel.

Render free services may sleep after inactivity and cold-start on the next request. Free Render disks are ephemeral: rule edits can be lost on redeploy/restart, even though the application writes SQLite. Export JSON for backup; paid persistent storage is required for guaranteed persistence there. The committed seed always recovers a valid library.

## Environment and storage

- `PORT`: 3000; binds to `0.0.0.0`.
- `PUBLIC_ORIGIN`: optional allowed browser origin. Empty defaults to same-origin hosting.
- `DEV_PASSPHRASE`: local default `local-mao`; must change for public use.
- `DATA_DIR`: `./storage`; contains `mao.sqlite` with library, selections and signing secret.

The SQLite file is implemented with sql.js and flushed after every configuration write. Keep one process/instance only. A persistent volume preserves the signing key and rule edits, not rooms. Back up SQLite or use JSON exports. Socket.IO heartbeat: 20 seconds, timeout: 60 seconds.
