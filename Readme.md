# AI Runtime — Local Deployment

Minimal local deployment for the AI Runtime stack (InvokeAI + Open Web UI + nginx + n8n). This repo provides docker-compose orchestration, configs, and example settings to run the services locally with GPU support.

## Contents
- [docker-compose.yaml](docker-compose.yaml) — service definitions (nginx, open-webui, invokeai, n8n, etc.)
- [invokeai/invokeai.example.yaml](invokeai/invokeai.example.yaml) — example InvokeAI configuration (see [`host`](invokeai/invokeai.example.yaml), [`port`](invokeai/invokeai.example.yaml), device and model dirs)
- [invokeai/invokeai.yaml](invokeai/invokeai.yaml) — your runtime config (created by you)
- [nginx/README.md](nginx/README.md) — nginx config notes and SSL instructions
- [nginx/nginx.conf](nginx/nginx.conf) and [nginx/conf.d/default.conf](nginx/conf.d/default.conf) — reverse-proxy setup
- [.env_sample](.env_sample) — example environment variables; copy to [.env](.env)

## Quick start
1. Copy environment sample and edit:
```sh
cp .env_sample .env
# Edit .env to set HUGGINGFACE_HUB_TOKEN, DB_POSTGRESDB_HOST, OLLAMA_BASE_URL, etc.
```
2. Review invokeai config or copy example:
```sh
# tweak only values you need; see keys such as `host` and `port`
cp invokeai/invokeai.example.yaml invokeai/invokeai.yaml
```
3. Start services:
```sh
docker-compose up -d
```
See service definitions in [docker-compose.yaml](docker-compose.yaml).

## Important configuration notes
- GPU support: containers declare `devices: - nvidia.com/gpu=all` in [docker-compose.yaml](docker-compose.yaml). Ensure NVIDIA Container Toolkit is installed.
- InvokeAI defaults: models and outputs are stored under `invokeai/` (see `models_dir`, `outputs_dir` in [invokeai/invokeai.example.yaml](invokeai/invokeai.example.yaml)).
- n8n database: n8n is configured to use Postgres via `DB_POSTGRESDB_HOST` (set in [.env](.env) and referenced in [docker-compose.yaml](docker-compose.yaml)).

## Networking & ports
- nginx listens on host port 80 (mapped in [docker-compose.yaml](docker-compose.yaml)) and proxies to open-webui / invokeai. See [nginx/conf.d/default.conf](nginx/conf.d/default.conf) and [nginx/README.md](nginx/README.md) for HTTPS setup.
- open-webui is exposed internally and storage is mounted via named volume (see [docker-compose.yaml](docker-compose.yaml)).

## Volumes & data
- invokeai data is mounted from the repo into the container:
  - local folder `invokeai/` -> `/invokeai` (see [docker-compose.yaml](docker-compose.yaml))
- Persistent named volumes declared in [docker-compose.yaml](docker-compose.yaml) are used for open-webui data.

## Troubleshooting
- If services fail to start, check logs:
```sh
docker-compose logs -f <service>
```
- GPU not visible: verify NVIDIA drivers and Docker runtime; check `nvidia-smi` on host.
- SSL: place certs in `nginx/ssl/` and follow [nginx/README.md](nginx/README.md) instructions.

## Where to edit runtime settings
- InvokeAI: edit [invokeai/invokeai.yaml](invokeai/invokeai.yaml) (see example file [invokeai/invokeai.example.yaml](invokeai/invokeai.example.yaml) for keys like [`host`](invokeai/invokeai.example.yaml) and [`port`](invokeai/invokeai.example.yaml)).
- nginx: modify configs in [nginx/conf.d/default.conf](nginx/conf.d/default.conf) or main [nginx/nginx.conf](nginx/nginx.conf).

## Useful files
- [docker-compose.yaml](docker-compose.yaml)
- [.env_sample](.env_sample) and [.env](.env)
- [invokeai/invokeai.example.yaml](invokeai/invokeai.example.yaml)
- [nginx/README.md](nginx/README.md)

Keep configs small and only override keys you need. For InvokeAI-specific options, consult the keys inside [invokeai/invokeai.example.yaml](invokeai/invokeai.example.yaml).