# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

CorridorKey is a neural green/blue-screen keying engine for VFX. Given an RGB plate plus a coarse alpha hint, it predicts an _unmixed_ straight-color foreground (3ch) and a linear alpha (1ch) — reconstructing semi-transparent edges (hair, motion blur) rather than emitting a binary mask. The model is a `timm` Hiera backbone (`hiera_base_plus_224.mae_in1k_ft_in1k`) with its first conv patched to 4 channels (RGB + hint), plus a custom `CNNRefinerModule` that adds delta logits before the final sigmoid.

## Commands

This project uses **uv** for everything — never invoke `pip` or `python` directly; prefix with `uv run`.

```bash
uv sync --group dev          # install deps + dev tools (pytest, ruff)
uv run pytest                # run all tests (fast, no GPU/weights needed)
uv run pytest -v             # verbose
uv run pytest -m "not gpu"   # skip CUDA tests (this is what CI runs)
uv run pytest tests/test_color_utils.py::test_name   # single test
uv run ruff format --check   # CI gate
uv run ruff check            # CI gate (rules: E,F,W,I,B; line length 120)
uv run corridorkey --help    # CLI smoke test (CI runs this)
```

CI (`.github/workflows/ci.yml`) runs `ruff format --check`, `ruff check`, then `pytest -m "not gpu"` across Python 3.10–3.13. Run all three locally before pushing.

`uv sync` extras are mutually exclusive (enforced in pyproject): `--extra cuda`, `--extra mlx`, `--extra rocm`. Base sync is CPU/MPS.

### macOS gotcha

Running pytest on Apple Silicon rewrites `uv.lock` with mac-specific markers. **Do not commit it** — run `git restore uv.lock` before staging.

## Architecture & layering

Three CLI/library layers sit on top of one inference engine. Know which one you're touching:

- **`corridorkey_cli.py`** — the `corridorkey` console entry point (Typer + Rich). Thin wrapper; calls `setup_rocm_env()` _before any torch import_, then delegates to `clip_manager.py`.
- **`clip_manager.py`** — the legacy argparse wizard **and** the importable pipeline library (`InferenceSettings`, `run_inference`, `generate_alphas`, `scan_clips`). This is where directory scanning, user prompting, and frame-by-frame export live. Importable headless for Nuke/Houdini/batch use.
- **`backend/`** — a newer service layer ("ez-CorridorKey", `CorridorKeyService` in `backend/service.py`) with a GPU job queue, project/clip state, and frame I/O. Used by the GUI front-end. `clip_manager.py` reuses a few of its helpers (`backend.frame_io`, etc.), but the two are largely parallel pipelines — don't assume a change in one propagates to the other.
- **`CorridorKeyModule/`** — the model. `CorridorKeyEngine` (`inference_engine.py`) loads weights and exposes `process_frame(rgb, hint, input_is_linear=...)`. `core/model_transformer.py` is the architecture (inference-only — training logic was deliberately stripped). `core/color_utils.py` is the compositing math.

`device_utils.py` centralizes device selection (`CUDA > MPS > CPU`); `CorridorKeyModule/backend.py` selects the inference backend (Torch vs MLX). Resolution order for both: CLI flag > env var (`CORRIDORKEY_DEVICE` / `CORRIDORKEY_BACKEND`) > auto-detect.

### Alpha-hint generators (optional, heavy, vendored)

`gvm_core/`, `VideoMaMaInferenceModule/`, `BiRefNetModule/` generate the coarse hint when the user doesn't supply one. `gvm_core/` and `VideoMaMaInferenceModule/` are **third-party research code** — excluded from ruff lint/format and from coverage; keep them close to upstream. They're invoked via `clip_manager.py --action generate_alphas`.

## Color math — the #1 source of bugs

Read `docs/LLM_HANDOVER.md` before touching anything color-related. Invariants:

- Model I/O is strictly `[0.0, 1.0]` float tensors. Input is assumed **sRGB**. Predicted FG (`res['fg']`) is **sRGB straight color**; predicted alpha (`res['alpha']`) is **linear**.
- The `Processed` EXR pass = `srgb_to_linear(fg)` premultiplied by linear alpha, saved as half-float. EXRs are linear, premultiplied.
- Use the **piecewise sRGB transfer functions** in `color_utils.py` (`srgb_to_linear`), never a plain gamma 2.2 curve. "Crushed shadows" / "dark fringes" complaints almost always trace to an sRGB↔linear step in the wrong order.
- The engine processes at a fixed **2048×2048** (Lanczos4 resize in/out); it's trained only on that resolution.

`OPENCV_IO_ENABLE_OPENEXR=1` must be set before any `cv2` EXR call — `clip_manager.py` sets it at import, and pytest sets it via `[tool.pytest.ini_options].env` (needs the `pytest-env` plugin, in the dev group).

## Output passes (per shot)

`/Matte` (linear alpha EXR), `/FG` (straight sRGB FG EXR — convert to linear before compositing), `/Processed` (linear premultiplied RGBA EXR), `/Comp` (8-bit PNG preview over checkerboard).

## Models & weights

Checkpoints live in `CorridorKeyModule/checkpoints/` and are **not** in git. The green checkpoint (`CorridorKey_v1.0.safetensors`) auto-downloads on first run; the blue one (`CorridorKeyBlue_1.0.safetensors`) downloads on demand when `--screen-color blue` or auto-detection picks blue. `.safetensors` is preferred over legacy `.pth` (no pickle). Screen color is per-clip: `--screen-color auto` samples the first frame's background pixels; `green`/`blue` force a checkpoint. Use `scripts/convert_pth_to_safetensors.py` to republish Torch weights.

## Clip input layout (easy to get wrong)

Per shot folder under `ClipsForInference/`, the input must be one of two shapes — `ClipEntry.find_assets()` / `organize_target()` in `clip_manager.py` decide which:

- **Video** → a single file in the **shot root** named `Input.<ext>` (e.g. `ShotName/Input.mp4`). A loose video dropped in the shot is auto-renamed to `Input.ext`.
- **Image sequence** → frames **inside** an `Input/` subfolder (`ShotName/Input/0001.png …`).

The trap: putting a video file _inside_ `Input/` makes the scanner treat that folder as an (empty) image sequence → "Input asset has 0 frames or could not be read." When staging clips by hand for headless runs, respect this; the interactive wizard / drag-and-drop organizes it for you. Alpha hints mirror the input shape under `AlphaHint/`, and input vs. alpha **frame counts must match** (`validate_pair`). Outputs land in a **per-shot** `Output/` subfolder (`ShotName/Output/{Matte,FG,Processed,Comp}/`), **not** the top-level `Output/`.

## Running locally on Apple Silicon (MPS)

`uv sync` (base) gives MPS-capable Torch. Always export `PYTORCH_ENABLE_MPS_FALLBACK=1` — some ops aren't implemented on MPS and will otherwise hard-error. Both video and image-sequence inputs decode via `cv2.VideoCapture` in `clip_manager` (the `ffmpeg_tools` subprocess path is used by `backend/`, not the CLI). The `objc[...] libavdevice ... implemented in both` warning at startup (cv2 vs av wheels) is noisy but benign — it does **not** break decoding.

**BiRefNet** (lightweight alpha-hint generator) is **only** wired into the interactive wizard — the `generate-alphas` CLI subcommand is GVM-only. For headless BiRefNet, import `run_birefnet` from `clip_manager` directly. BiRefNet loads via `trust_remote_code=True` (its HF repo ships a custom architecture) and runs fp32 on MPS (fp16 NaNs on Apple's backend); see `BiRefNetModule/wrapper.py`.
