# OpenMT2

Open-source, cross-platform reimplementation of the Metin2 client, with the
proprietary runtime dependencies replaced by clean-room equivalents. The client
builds on Linux and cross-compiles for Windows from the same codebase.

**Documentation, specifications and project status → <https://open-mt2.github.io/docs/>**

## Repositories

| Repository | Purpose |
| :--- | :--- |
| [`client`](https://github.com/open-mt2/client) | Metin2 game layer: GameLib, UserInterface and `metin2client` |
| [`neoeter`](https://github.com/open-mt2/neoeter) | Engine: platform, IO, image, render, audio, model, terrain and effects |
| [`neoeter-python`](https://github.com/open-mt2/neoeter-python) | Embedded CPython 2.7 host and Python bindings |
| [`devkit`](https://github.com/open-mt2/devkit) | Docker devkit: toolchain base image, client image, build/packaging scripts and the vendored tree geometry cache |
| [`speedtreebkr`](https://github.com/open-mt2/speedtreebkr) | Offline regeneration tool for the tree geometry cache |
| [`docs`](https://github.com/open-mt2/docs) | Sources of the documentation site |
| [`devkit.nix`](https://github.com/open-mt2/devkit.nix) | Nix development kit |
| [`agent-docs`](https://github.com/open-mt2/agent-docs) | AI-agent guidance, skills and issue-review archive |

## Disclaimer

Personal sandbox project exploring agentic AI on a large codebase. It is not
intended to become stable production software, and AI-assisted code analysis and
refactoring are assumed throughout. See the documentation for the current state
and the roadmap.
