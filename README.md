# workflow

Custom AI workflow editor built around the Comfy-Org ecosystem.
Original code will live in [editor/](editor/) — all submodule directories are upstream read-only.

[Self-Hosting Comfy](https://claude.ai/share/9d1e0f23-9dd0-4b31-8908-1b9c88db012c) guidance provided by Claude.ai

## Get keys for .env

#### Clerk keys for comfyui-deploy

Get both keys free at [dashboard.clerk.com](https://dashboard.clerk.com) → create an account → **Create application** → **Configure** → **API Keys**. Add both to `docker/.env` in the webroot root:

- `CLERK_SECRET_KEY` — starts with `sk_test_` or `sk_live_` (Secret keys section)
- `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` — starts with `pk_test_` or `pk_live_` (Publishable key section)

The `start chat` command checks for `CLERK_SECRET_KEY` before starting comfyui-deploy; without it the Next.js middleware crashes on startup.

## Local Servers

| Service | URL |
|---------|-----|
| Workflow editor (via chat server) | http://localhost:8888/workflow/editor/ |
| LLM API key setup | http://localhost:8888/chat/keys/ |
| ComfyUI Deploy dashboard (requires Clerk keys) | http://localhost:3001 |
| ComfyUI UI (Python, when running) | http://localhost:8887 |
| ComfyUI API | http://localhost:8887/api/ |
| ComfyUI model browser | http://localhost:8887/models |

See [AGENTS.md](AGENTS.md) for the start command and one-time build step.

## Model Providers & API Keys

ComfyUI workflows call external providers via API nodes — no local GPU needed.

| Resource | Link |
|----------|------|
| LLM API keys (our setup) | http://localhost:8888/chat/keys/ |
| API workflow bundles by provider | http://localhost:8888/workflow/comfy_templates/bundles.json |
| Supported local model architectures | http://localhost:8888/workflow/comfyui/comfy/supported_models.py |
| Local model path config | http://localhost:8888/workflow/comfyui/extra_model_paths.yaml.example |
| Comfy platform key (stored in ComfyUI UI) | http://localhost:8887 (when running) |

`bundles.json` lists all template workflows grouped by provider: `api_bria_*`, `api_bytedance_*`, `api_kling_*`, `api_vidu_*`, `api_meshy_*`, `media-api`, and more.

## Remote Node Protocol

[comfy-remote-nodes](comfy-remote-nodes/) — fetches node descriptors from an **RNP/1** server and exposes them as native ComfyUI nodes. Decouples compute from the UI layer; any provider-agnostic RNP/1 server can publish nodes.

Configure before starting ComfyUI:
```
RNP_SERVER_URL=http://your-rnp-server:9190
```
Unset = zero remote nodes registered; ComfyUI starts normally.

## Comfy Submodules

### [comfyui](comfyui/) — Core engine
Python backend. Node graph execution, model management, API server.
- main.py — server entry point
- api_server/ — REST + WebSocket API
- comfy/cli_args.py — startup flags (--port, --cpu, etc.)

### [comfyui_frontend](comfyui_frontend/) — Vue frontend
Browser-side graph UI rendered by the Python server.
- src/views/GraphView.vue — main node canvas
- src/views/LinearView.vue — linear workflow view
- src/router.ts — page routing
- src/components/ — UI components

### [comfyui-manager](comfyui-manager/) — Extension manager
Install, remove, and enable/disable custom nodes from the ComfyUI UI.
- __init__.py — manager entry point
- custom-node-list.json — node registry

### [comfyui-deploy](comfyui-deploy/) — Deployment platform
Serverless GPU deployment and workflow API infrastructure → [http://localhost:3001](http://localhost:3001)
- example_workflows/ — workflow JSON examples
- comfy-nodes/ — deploy-specific custom nodes
- web/ — Next.js dashboard (requires Clerk keys — see below)

### [comfy-cli](comfy-cli/) — CLI tool
Install and manage ComfyUI from the terminal.
- comfy_cli/ — CLI source
- README.md — usage and installation guide

### [comfy_templates](comfy_templates/) — Workflow templates
Official workflow templates and reusable subgraph blueprints.
- blueprints/ — reusable node components
- [bundles.json](http://localhost:8888/workflow/comfy_templates/bundles.json) — template bundle manifest (API provider index)
- input/, output/, thumbnail/ — example images (large; ~2.5 GB locally)

### [comfy-remote-nodes](comfy-remote-nodes/) — Remote Node Protocol client
Exposes RNP/1 server nodes inside ComfyUI. Set `RNP_SERVER_URL` to connect.
- protocol.py — RNP/1 handshake and envelope codec
- proxy_node.py — V3 node class factory
- registry.py — dynamic node registration
- client.py — HTTP client (sync execute, async poll, presigned upload)

## Editor (our code)

[editor/](editor/) — All original work: custom pages, flowcharts, and wrapper UI
that ingests the Comfy submodules to guide stages of AI pipelines for image,
video, and other automated content creation.

See [AGENTS.md](AGENTS.md) for development guidance and constraints.

[neilsonnn/image-blaster](https://github.com/neilsonnn/image-blaster)
Creates 3D environments, SFX, and meshes from a single image using Claude skills, World Labs, and FAL.
