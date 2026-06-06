# Open Issues & PRs

Snapshot pulled 2026-06-06. **64 open issues**, **15 open PRs**.

## Open PRs (15)

| #   | Title                                                   | Author        | Notes                  |
| --- | ------------------------------------------------------- | ------------- | ---------------------- |
| 251 | Introduce new static code analysis tools                | zalox         |                        |
| 250 | INT8 ONNX quantization for edge (Pi / low-spec CPU)     | shahwork005   |                        |
| 240 | Fix checkpoint path bug + mask channel inconsistency    | kyroX-D       | bug fix                |
| 236 | Retire argparse CLI in favor of `corridorkey`           | mountarreat   | ties to issue #26      |
| 234 | Route engine creation through `create_engine` factory   | mountarreat   |                        |
| 233 | Default unset optional CLI flags to `InferenceSettings` | mountarreat   |                        |
| 232 | Set `OPENCV_IO_ENABLE_OPENEXR` in conftest              | mountarreat   | EXR/pytest harness fix |
| 221 | Unblock benchmark harness + perf PR template            | apekshik      |                        |
| 187 | Automatic MLX selection and setup                       | michal1000w   |                        |
| 185 | Automatic weights download                              | michal1000w   |                        |
| 177 | Test: clean matte boundaries                            | rocketmark    |                        |
| 168 | Fix clip scan dedup by display name                     | justingleader |                        |
| 149 | Inference testability + `--start-frame` support         | karimnagdii   |                        |
| 124 | Multithreaded I/O for high-core-count CPUs              | SpaceMarty    |                        |
| 54  | VRAM & inference optimizations (CLI flags)              | cmoyates      | **DRAFT**              |

> **mountarreat** has a 4-PR cleanup suite (232/233/234/236) — likely the easiest low-risk batch to review together.

## Open Issues (64)

### Bugs

- #238 [Bug]: (no detail)
- #231 "GPU/Hardware (Optional)" field in bug template is actually mandatory
- #230 ERROR text incorrect when running BiRefNet General-Lite
- #190 Vertical Video Workflow Broken
- #184 Inconsistent quality
- #161 Not working on Linux with RTX 5050 8GB
- #158 Output is all black
- #145 `'TLE' & 'ho' not recognized` (broken launcher script)
- #123 `error: Failed to spawn: corridorkey`
- #97 WinError 267 - Invalid directory name when running locally
- #44 `error: typing_extensions`
- #9 Problem with preinstalled python, RTX 5090, drag-and-drop

### Licensing / policy

- #229 "This is not an open source project"
- #68 [FR]: `LICENSE` meta-discussion

### Color math (matches CLAUDE.md warning)

- #28 Track: gamma 2.2 vs piecewise sRGB inconsistency
- #191 Textures sometimes get lost on semi-transparent stuff
- #74 Avoiding the Despill Step

### Platform / packaging

- #224 Proper packaging for Linux, Mac, Windows
- #218 [Suggestion] Intel Mac support
- #201 Permission denied macOS on network
- #87 Docker Setup + GUI
- #83 Request: Docker Image?
- #69 AMD GPU support?
- #41 running on macOS

### Feature requests

- #214 Image support
- #195 Support for non-green chroma key (blue, magenta)
- #193 Tune models based on log & linear sources?
- #192 HuggingFace Demonstration Space?
- #180 Garbage Mask / ProcessingBox for speed gain
- #146 HDR support?
- #127 VideoMaMa tiling
- #106 Quick Suggestion
- #105 Possible alternate method
- #103 SAM (Segment Anything) Integration?
- #84 Suggestion: infer normals and depth
- #75 ComfyUI Version planned?
- #46 After Effects Plugin
- #219 Lateral idea: use with sam3.1
- #208 [Stupid Idea]: Gaussian Splats for realistic images without alpha
- #113 [Improvement] Potential ways to improve quality further

### Dataset / training

- #167 Dataset release (please)
- #129 Possible improvements on the data augmentation pipeline
- #94 Request: training dataset for model pruning
- #86 Question about raw dataset / data generation pipeline
- #77 corridorKey model architecture

### Acknowledgements / docs

- #147 VideoMaMa acknowledge
- #71 [Proposal] Free Documentation Website via GitHub Pages
- #56 Unclear what files should be named to be seen by the program

### Internal refactor / test tracking (maintainer-filed)

- #252 Pressing code changes on `CorridorKeyModule`
- #246 chore(test): validate estimate_screen_color thresholds against real plates
- #245 chore(refactor): unify the two ClipEntry implementations
- #244 feat(mlx): support CorridorKeyBlue on the MLX backend
- #181 refactor(clip_manager): move videomama loader inside a safer loop
- #27 VideoMaMa: run_videomama loads all frames into memory before chunking
- #26 Verify corridorkey entry point works end-to-end
- #25 Refactor clip_manager.py: extract pipeline orchestration from file I/O
- #21 CI: Expand test matrix, add macOS/Windows runners
- #20 Testing: Make mock engine pattern more maintainable
- #18 Replace print() with logging in model_transformer.py / inference_engine.py
- #15 Launcher scripts should check for uv before running
- #14 Core pipeline function run_inference() has zero test coverage
