# AGENTS.md

The submodule directories in this repo (comfyui, comfyui_frontend, comfyui-manager,
comfyui-deploy, comfy-cli, comfy_templates, comfy-remote-nodes) are upstream
dependencies and must not be edited here. Pull updates from their respective sources
using `git submodule update --remote`.

Our custom interface and all original work resides in the `editor` directory.

## No Local LLMs

**Never run LLM inference locally.** All model calls go through external APIs.
API keys are configured at **http://localhost:8888/chat/keys/** and stored in local
browser cache or pulled from `docker/.env`. No local model checkpoint files are needed.

Always start ComfyUI with `--cpu` to disable GPU and local model loading.

## Start Server (Workflow Context)

ComfyUI serves as the web server on port 8887 — the same port used by the rest of the
webroot. See **Option 3** in the `Start HTTP Server` section of `team/AGENTS.md` for
the full command.

Quick reference (run from the webroot root folder):

```bash
lsof -ti:8887 > /dev/null 2>&1 || \
  nohup workflow/env/bin/python3 workflow/comfyui/main.py \
    --port 8887 --cpu > workflow/comfy.log 2>&1 &
```

Verify: `curl -s http://localhost:8887/system_stats && echo "ComfyUI OK"`

## Remote Node Protocol (comfy-remote-nodes)

`comfy-remote-nodes` lets ComfyUI nodes run on a remote RNP/1 server — the primary
path for decoupling compute from the UI and routing execution through external APIs
rather than local GPU.

Set `RNP_SERVER_URL` before starting ComfyUI. If unset, zero remote nodes are
registered and ComfyUI starts normally with no error.

```bash
export RNP_SERVER_URL=http://your-rnp-server:9190
```

To install into a live ComfyUI instance instead of using the submodule pointer:
```bash
# Symlink from the submodule (avoids a second git clone)
ln -s $(pwd)/workflow/comfy-remote-nodes \
      workflow/comfyui/custom_nodes/comfy-remote-nodes
```

The extension handles:
- RNP/1 manifest + capability handshake, ETag conditional GETs
- Sync `execute` and async `execute_async` + poll/cancel
- Image / mask / audio / video envelope encode + decode
- `TASK_LOST` detection with idempotent resubmission
- `SERVER_BUSY` / `MAINTENANCE` 503 envelopes with `Retry-After` honoring

## Build (one-time setup)

Install Python dependencies into `workflow/env/` before the first server start.
**Run this separately — it is NOT part of the start server command.**

```bash
workflow/env/bin/pip install torch --index-url https://download.pytorch.org/whl/cpu
workflow/env/bin/pip install av --only-binary=:all:
workflow/env/bin/pip install -r workflow/comfyui/requirements.txt \
  --extra-index-url https://download.pytorch.org/whl/cpu
```

The `workflow/env/` venv was created from Anaconda Python 3.12. Run from the webroot
root folder. `av` (video support) requires `pkg-config` + FFmpeg dev headers to build
from source; `--only-binary=:all:` installs a binary wheel if available and skips
video support if not (`pkg-config` can be installed via `brew install pkg-config ffmpeg`).

**Intel Mac (x86_64) note**: PyTorch dropped pre-built CPU wheels for macOS x86_64
after 2.2.2. The current comfyui submodule requires torch 2.4+ (`torch.library.custom_op`,
`torch.serialization.add_safe_globals`). Until a compatible wheel or Docker path is
confirmed, the quickstart.sh HTTP server (Option 2 in team/AGENTS.md) serves port 8887
as a fallback for local development. The comfyui-deploy submodule provides a hosted
GPU path via [comfydeploy.com](https://comfydeploy.com) for full execution.

## Repo Size

The workflow repo pushes only submodule commit pointers — not submodule content.
The GitHub push stays well under 1 MB regardless of local working-directory size.
The 9+ GB local footprint comes from:

| Source | Local size | Push to GitHub |
|--------|-----------|---------------|
| comfy_templates/ (input, output, thumbnail) | ~2.5 GB | 0 (submodule pointer) |
| comfyui-deploy/web/node_modules | ~851 MB | 0 (submodule pointer) |
| env/ (Python venv) | ~1.6 GB | 0 (gitignored) |
| comfyui_frontend git objects | ~611 MB | 0 (submodule pointer) |

To reduce local disk use, fetch submodules with `--depth 1` (shallow clone):
```bash
git submodule update --init --depth 1
```

For `comfy_templates` you can also do a sparse checkout to skip input/output:
```bash
git -C workflow/comfy_templates sparse-checkout set \
  templates blueprints bundles.json packages scripts
```

## Editor Development

The `editor/` directory is where all original code lives. Use it to:

- Add pages that wrap or visualize ComfyUI workflows
- Build flowchart UIs that guide stages of AI image/video creation pipelines
- Inject our code around untouched Comfy submodule functionality
- Create interfaces for automated actions (batch processing, API chaining, etc.)

**Never import or modify submodule files directly.** Ingest them by referencing their
API endpoints (http://localhost:8887) or by reading their JSON workflow files.

When adding a new page in `editor/`, use the localsite template:
```
https://raw.githubusercontent.com/ModelEarth/localsite/refs/heads/main/start/template/index.html
```

## Submodule Reference

| Directory | Upstream Repo | Purpose |
|-----------|--------------|---------|
| comfyui/ | github.com/Comfy-Org/ComfyUI | Python backend and execution engine |
| comfyui_frontend/ | github.com/Comfy-Org/ComfyUI_frontend | Vue.js browser UI |
| comfyui-manager/ | github.com/Comfy-Org/ComfyUI-Manager | Custom node management |
| comfyui-deploy/ | github.com/BennyKok/comfyui-deploy | Serverless deployment platform |
| comfy-cli/ | github.com/Comfy-Org/comfy-cli | CLI installer and manager |
| comfy_templates/ | github.com/Comfy-Org/workflow_templates | Workflow and blueprint templates |
| comfy-remote-nodes/ | github.com/Comfy-Org/comfy-remote-nodes | RNP/1 remote node client |
