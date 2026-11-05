# RunPod ComfyUI Serverless Template

A reusable template for deploying ComfyUI workflows on RunPod Serverless.

The runtime core stays the same from project to project. Everything that changes per deployment (custom nodes, models, the workflow and test payloads) lives in a few config files, so a new endpoint is mostly a config change, not a code change.

## What Stays Stable

- `handler.py`: RunPod handler contract and the ComfyUI execution loop (queue, websocket progress, history, output collection, S3 upload)
- `src/start.sh`: startup orchestration and model path detection
- `Dockerfile`: image build for custom nodes and runtime tools
- `scripts/`: model download, node install, volume and readiness checks

## What You Change Per Project

- `project-config/custom-nodes.txt`: registry or git custom nodes
- `project-config/model-manifest.txt`: models to place on the Network Volume
- `project-inputs/workflow-api.json`: the workflow exported in API format
- `project-inputs/test-payload-*.json`: payloads for local and endpoint tests
- `.runpod/hub.json`: endpoint environment variables

## Repository Layout

```text
.runpod/           RunPod Hub metadata and env definitions
project-config/    custom nodes, model manifest, env example
project-inputs/    workflow and test payloads
scripts/           model download, node install, readiness checks
src/               container start script
tests/             handler unit tests
Dockerfile
docker-compose.yml
handler.py
QUICK_START.md
```

## Recommended Operating Model

- Install custom nodes into the image.
- Put models on a Network Volume, not in the image.
- Keep workflow-specific logic in the workflow JSON before touching `handler.py`.
- Run `scripts/verify-project-readiness.sh` before building, and `scripts/verify-volume-models.sh` before creating the endpoint.

Start with `QUICK_START.md`.

## Request Contract

```json
{
  "input": {
    "workflow": {},
    "images": []
  }
}
```

`images` is optional. Each image can be base64 data or an `http(s)` URL.

## Response Contract

Read generated media from `output.images[]`. Each item contains `filename`, `type` and `data`. When S3-compatible storage is configured (`BUCKET_ENDPOINT_URL` and keys), `data` is a URL; otherwise it is base64 and may already include a data URI prefix. Returning URLs avoids payload size limits on large images and videos.

## Notes

Production runbooks, pitfall notes and delivery checklists from real deployments are kept private. Happy to walk through them on request.

## Credits and License

`handler.py` started from [runpod-workers/worker-comfyui](https://github.com/runpod-workers/worker-comfyui), licensed under AGPL-3.0. This template keeps that license (see `LICENSE`). Changes in this repository include S3-compatible region detection, URL image inputs, quieter numba logging, model path detection at startup, and the config-driven project layout and scripts above.
