<div align="center">
  <img
    src="https://raw.githubusercontent.com/labeloo/.github/main/profile/assets/labeloo-hero.png"
    alt="Labeloo — AI Annotation Platform"
    width="100%"
  />
</div>

# Labeloo

**Image & video annotation platform.** Click an object and it is segmented; click
it once in a clip and it is tracked to the end. Teams, roles and task assignment;
YOLO/COCO export; and train your own model from the annotations you make.

Open-core and dual-mode: self-host the whole thing with one `docker compose up`,
or use the hosted cloud service. Same codebase, switched by one flag. Core is
**Apache-2.0**.

## Repositories

| Repo | What it is | Stack |
|------|-----------|-------|
| [**platform**](https://github.com/labeloo/platform) | Start here. docker-compose, `.env` template, setup script, docs — ties the services together. | Compose |
| [**frontend**](https://github.com/labeloo/frontend) | Annotation UI: Konva canvas, magic stick, video tracking, projects, teams, admin. | Nuxt 4 · Vue 3 · Tailwind |
| [**backend**](https://github.com/labeloo/backend) | API: auth, projects/tasks, the compute **broker**, training orchestration, billing, admin. Runs on Node (libsql) or Cloudflare (D1). | Hono · Drizzle |
| **sam2-service** *(private for now)* | Segmentation & video object tracking (the magic stick). | FastAPI · SAM 2 · PyTorch |
| **trainer-service** *(private for now)* | Trains YOLO models from completed annotations. | FastAPI · ultralytics |

## Run it locally

```bash
git clone https://github.com/labeloo/platform labeloo && cd labeloo
./setup.sh                 # clones the four service repos as siblings
cp .env.example .env       # then set JWT_SECRET (openssl rand -hex 32) and ADMIN_PASSWORD
docker compose up --build  # NVIDIA GPU: add  --profile gpu
```

Open <http://localhost:3000> and sign in with the admin account from `.env`.

## How SAM 2 is set up

The browser never calls SAM 2 directly. Each organization registers its SAM 2
service under **Settings → Connections** (URL `http://sam2:8000` on the compose
network, optional API key); the backend proxies every segmentation call through
its **broker**, so the key stays server-side and usage is metered.

The SAM 2 checkpoint is **not** bundled — it downloads on the first request into a
named volume (`SAM2_MODEL=tiny|small|base_plus|large` picks the size). The first
magic-stick click takes a minute or two; every later one is fast. On a GPU, run
`docker compose --profile gpu up` for roughly an order-of-magnitude speedup on
video tracking.

Model training works the same way: a `trainer` connection (`http://trainer:8020`)
per organization, used from each project's **Models** section.

## Hosted / cloud

The backend also runs on Cloudflare Workers (D1 + R2); set `EDITION=cloud` to turn
on plans, quotas and the admin panel. Leave it unset for the unlimited community
edition.
