# home-stacks

A monorepo of self-hosted services, each defined as a **Docker Compose stack** in its own
top-level folder (e.g. `immich/`, `n8n/`, `media/`). Stacks are deployed/managed via
Dockge / Portainer on a home server. There is no application source to build here (with a
few exceptions, see "Custom-built images") — this repo is **infrastructure-as-config**.

## Repository layout

- `<service>/docker-compose.yml` — the stack definition (required).
- `<service>/stack.env` — env vars for the stack, referenced via `env_file: - stack.env`.
  Often present even if empty. Values use the `VAR=${VAR:-default}` indirection pattern so
  the real values come from the host environment / secret manager, not from git.
- `<service>/Dockerfile` (+ extra files) — only for the few stacks whose image we build ourselves.
- `TEMPLATE/` — the starting point for a new stack. Copy it and fill in the placeholders.
- `.github/workflows/` — CI that builds & pushes our custom images to GHCR.

## Conventions for a stack's docker-compose.yml

Follow `TEMPLATE/docker-compose.yml`. Key rules observed across existing stacks:

- **`name:`** the stack, and give each service a `container_name:` (kebab or snake case,
  matching what the upstream project uses).
- **Volumes / persistent data** live under `/DATA/AppData/<service>/...` on the host
  (this server uses the CasaOS `/DATA/AppData` layout). Bind-mount config/data there, e.g.
  `- /DATA/AppData/<service>/config:/config`. Prefer this over named volumes.
- **`env_file: - stack.env`** for configuration. Put secrets/values as `${VAR}` references
  in `stack.env`, never hardcoded in the compose file.
- **`restart: unless-stopped`** for app services (databases sometimes use `restart: always`).
- **Networking**: most stacks use `network_mode: bridge`. Multi-service stacks instead
  define a named bridge network (e.g. `networks: { n8n: { name: n8n, driver: bridge } }`).
  Some stacks attach to a pre-existing `external: true` network (`media`, `vpn`).
- **Timezone** is `Asia/Ho_Chi_Minh` (set via `TZ` env when the image needs it).
- **Pin image versions** through an env var with a sensible default, e.g.
  `image: .../n8n:${N8N_IMAGE_VERSION:-latest}`. Several official upstream files also pin
  by digest (`@sha256:...`) — preserve those when updating.
- LinuxServer-style images take `PUID` / `PGID` / `TZ` via the `environment:` block.

## Host port allocation — IMPORTANT

Every published host port must be **globally unique across all stacks** to avoid collisions
on the shared host.

- **Default range**: `111xx` — start at `11111` and go up. Pick the next free number.
- **Media stack**: uses the `333xx` range (reserved for media-related services only).

The **authoritative, up-to-date port registry** is the Trilium note at:
`https://note.zum.vn/share/WHSjF3nSJ4YY`

**Always check that note before allocating a new port.** Do not rely solely on grepping the
repo, as the note may list reservations that aren't yet committed. After allocating, the note
should be updated to reflect the new assignment.

Map the chosen host port to the container's internal port in the compose file:
`- "1111x:<container-port>"`

## Secrets

- `.env` and `meshcentral/config.json` are gitignored — **never commit secrets**.
- `stack.env` holds **references** (`${VAR}`), not real values; the host/secret manager
  (Infisical is used here) injects the actual values at deploy time.

## Custom-built images

A few stacks build their own image instead of pulling upstream (e.g. `crontab-guru/`,
`copilot-api/`, custom n8n runners). These have a `Dockerfile` and a matching workflow in
`.github/workflows/` that builds multi-arch (`linux/amd64,linux/arm64`) and pushes to
`ghcr.io/<owner>/...`. Workflows trigger on `push` to `main` filtered by `paths:` for that
stack, plus a daily `schedule` and `workflow_dispatch`. The compose file then references the
GHCR image. If you change such a stack's build, update both the Dockerfile and its workflow.

## Adding a new stack (checklist)

1. `cp -r TEMPLATE <service>` and rename placeholders (`stack_name`, `image_url`,
   `container_name`, `app_name`, ports).
2. Set host volumes under `/DATA/AppData/<service>/`.
3. Choose a free, unique host port (see allocation rule above).
4. Move any configurable/secret values into `stack.env` as `${VAR}` references.
5. If the image is custom-built, add a `Dockerfile` and a `.github/workflows/` build job.

## Commit conventions

Short, lowercase, imperative messages scoped to the stack — e.g. `update media`,
`fix ports`, `media: add Watchtower for automatic image updates`. Keep changes scoped to one
stack per commit where practical.
