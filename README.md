# procgen-weights

Public hosting for model weight shards used by a Bittensor **Subnet 17 (404-GEN)** `threejs` competition miner.

The validator re-runs a candidate miner's Docker image on a fresh GPU pod for the verification audit
(`require_audit.json` -> `generated.json`). That image must be able to download its exact model weights
without any credentials or third-party hub. Every release in this repository is one immutable weight
snapshot: the image fetches the release matching the pinned tag, verifies every byte against
`manifest.json`, and serves the model from the local directory.

## Layout of a release

| asset | content |
|---|---|
| `manifest.json` | `files[]` (name, size, sha256), `shards[]` (name, size, sha256, byte-range members), `model_dir_sha256` tree hash. Uploaded last; its presence marks the release complete. |
| `<file>` | a model file small enough to be a single asset (config, tokenizer, processor files, small safetensors) |
| `<file>.partNNN` | contiguous byte range `[NNN*chunk, (NNN+1)*chunk)` of a large safetensors file (chunk = 1.9 GB) |

Nothing is re-encoded: concatenating the parts of a file in order reproduces the original file exactly.

## Fetching

```
python3 fetch-weights.py --tag <tag> --dest /path/to/model   # stdlib only, 8 parallel connections, resume + retry
```
or, with plain shell tools: `curl -L -o <asset> https://github.com/uiauuiuiiauo/procgen-weights/releases/download/<tag>/<asset>`
and `cat file.part000 file.part001 ... > file`, then compare `sha256sum` with `manifest.json`.

The fetch tool source is published in the miner repository next to the Dockerfile that uses it.

## Licensing

Weights are fine-tunes of `Qwen/Qwen3.5-27B`-family checkpoints; the upstream `LICENSE` and `NOTICE` files
are included in every release as regular assets.
