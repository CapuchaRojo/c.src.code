# Mission Read

This Phase 1 audit records what is presently in `c.src.code` based on direct repository inspection. The goal of this checkpoint is to inventory the visible repository surfaces and map how the major parts relate, without proposing changes, reproducing implementation logic, or planning follow-on patches.

# Observed Truth

- The checked-out repository root resolves to `C:/Users/mich3/GitHubProjects/c.src.code`.
- The root README identifies the project as Claw Code and states that the canonical implementation lives in `rust/`.
- The top-level tree contains a Rust workspace, a companion Python/reference workspace, supporting docs, CI workflow definitions, assets, and container/install surfaces.
- The root `git status --short` output at audit time showed one pre-existing untracked file: `AGENTS.md`.
- The Rust workspace is organized as a Cargo workspace with 9 crates under `rust/crates/`.
- The repository also contains a Python-side `src/` tree with 37 top-level `.py` files plus 30 package directories, along with one root test file in `tests/`.
- The checked-in docs and CI surfaces consistently treat `rust/` as the active runtime surface and `src/` plus `tests/` as a companion or reference-oriented surface.

# Inventory

## Top level

- Root docs: `README.md`, `USAGE.md`, `PARITY.md`, `PHILOSOPHY.md`, `ROADMAP.md`, `CLAUDE.md`, `AGENTS.md`
- Operational files: `Containerfile`, `install.sh`, `.gitignore`, `.claude.json`
- Main directories: `.github/`, `assets/`, `docs/`, `rust/`, `src/`, `tests/`

## Docs and governance

- `README.md` presents the repo entrypoint and points readers toward `USAGE.md`, `rust/README.md`, `PARITY.md`, and `docs/container.md`
- `PHILOSOPHY.md`, `ROADMAP.md`, and `PARITY.md` provide project framing and status documentation
- `docs/` currently contains one checked-in operational document: `container.md`

## CI and automation

- `.github/workflows/rust-ci.yml` defines documentation, formatting, clippy, and workspace test jobs
- `.github/workflows/release.yml` is present as an additional workflow surface
- `.github/scripts/check_doc_source_of_truth.py` enforces documentation branding/source-of-truth constraints

## Rust workspace

- Workspace root: `rust/Cargo.toml`, `rust/Cargo.lock`
- Crates present:
  - `api`
  - `commands`
  - `compat-harness`
  - `mock-anthropic-service`
  - `plugins`
  - `runtime`
  - `rusty-claude-cli`
  - `telemetry`
  - `tools`
- Rust-side docs/support files include `rust/README.md`, `rust/USAGE.md`, `rust/PARITY.md`, `rust/MOCK_PARITY_HARNESS.md`, `rust/mock_parity_scenarios.json`, and `rust/TUI-ENHANCEMENT-PLAN.md`
- Rust-side scripts present:
  - `rust/scripts/run_mock_parity_diff.py`
  - `rust/scripts/run_mock_parity_harness.sh`

## Python/reference workspace

- `src/` contains a Python workspace centered on inventory, manifesting, routing, parity-audit, and mirrored command/tool surfaces
- `src/reference_data/` contains command, tool, archive, and subsystem snapshot data
- Numerous package directories under `src/` appear to mirror subsystem names rather than define the primary runtime
- `tests/test_porting_workspace.py` exercises the Python-side workspace and its mirrored inventories

## Plugin and sample extension assets

- `rust/crates/plugins/bundled/` contains two bundled sample plugin roots:
  - `example-bundled`
  - `sample-hooks`
- Each bundled sample includes `.claude-plugin/plugin.json` and shell hook examples

## Assets

- `assets/` contains repository imagery and screenshots used for docs/presentation rather than runtime execution surfaces

# Architecture Map

The repository has two visible architectural layers.

First, `rust/` is the active product layer. The root docs, install flow, container docs, and CI all converge on the Rust workspace as the canonical implementation. Within that workspace, the `rusty-claude-cli` crate serves as the user-facing binary surface, while the other crates separate concerns such as runtime/session behavior, provider access, command metadata, built-in tools, plugins, telemetry, and compatibility testing.

Second, the root-level `src/` and `tests/` directories form a companion Python layer. This layer appears to function as a mirrored reference workspace for inventorying commands, tools, subsystem snapshots, parity auditing, and lightweight session/bootstrap simulation. The Python CLI advertises summary, manifest, parity-audit, command/tool listing, routing, bootstrap, turn-loop, and remote-mode style surfaces rather than the primary shipped CLI runtime.

At a high level, the Rust workspace is arranged around this dependency shape:

- `rusty-claude-cli` is the binary entrypoint and composes the other workspace crates
- `runtime` is the core coordination layer for config, session management, permissions, sandbox state, prompt loading, MCP lifecycle, remote/session support, hooks, and worker/task surfaces
- `api` provides provider-facing client, streaming, request/response, and auth-adjacent surfaces
- `tools` gathers built-in tool definitions and tool execution surfaces, and also carries runtime-facing discovery/search surfaces
- `commands` centralizes slash-command definitions and command rendering/help behavior
- `plugins` carries plugin metadata and install/enable/disable style lifecycle surfaces
- `telemetry` holds usage and trace-oriented types
- `mock-anthropic-service` and `compat-harness` support deterministic parity/testing workflows

This yields an overall architecture where the CLI crate fronts the system, the runtime crate holds the central operating model, the API/tools/commands/plugins crates extend that model in bounded ways, and the Python workspace preserves a separate mirrored audit/reference layer.

# Execution Surfaces

- Primary binary surface: `rust/crates/rusty-claude-cli/src/main.rs` builds the `claw` executable declared in `rust/crates/rusty-claude-cli/Cargo.toml`
- Rust workspace build/test surface: `cargo build --workspace`, `cargo test --workspace`, `cargo fmt`, and `cargo clippy` from `rust/`
- Installer surface: root `install.sh`
- Container surface: root `Containerfile` plus `docs/container.md`
- CI surface: `.github/workflows/rust-ci.yml`
- Rust mock/parity surface:
  - `rust/scripts/run_mock_parity_harness.sh`
  - `rust/scripts/run_mock_parity_diff.py`
  - `rust/crates/mock-anthropic-service/`
- Python companion CLI surface: `python -m src.main ...`
- Python validation surface: `tests/test_porting_workspace.py`

# Extension Points

- Plugin lifecycle surface in `rust/crates/plugins/` with bundled sample plugin directories under `rust/crates/plugins/bundled/`
- Hook-oriented extension samples in bundled plugin folders via `hooks/pre.sh` and `hooks/post.sh`
- MCP-related runtime surfaces are represented by dedicated runtime modules and command/help surfaces, indicating external server integration as a first-class extension seam
- Skills, agents, plugin, and MCP command surfaces are represented in the Rust command inventory, indicating user-visible extensibility and inspection paths
- The Python `src/reference_data/` snapshots provide a separate extension seam for reference inventories and mirrored workspace analysis rather than runtime execution

# Open Questions

- The repository root name is `c.src.code`, while the visible product/docs branding is Claw Code; this may be intentional, but the naming relationship is not explained in the inspected docs
- The exact release and packaging role of `.github/workflows/release.yml` was not reviewed in depth during this checkpoint
- The long-term role of the Python companion workspace relative to the Rust runtime is described at a high level in the root docs, but its maintenance boundary and source-of-truth expectations are not fully spelled out in one place
- The repo includes several planning/status documents (`ROADMAP.md`, `PARITY.md`, `TUI-ENHANCEMENT-PLAN.md`) whose governance relationship to the shipped runtime was not normalized in this audit

# Clean-Room Notes

- This report is inventory-only and architecture-only
- It was written from direct file and manifest inspection
- It does not reproduce code bodies, patch plans, or feature proposals
- It does not treat inferred behavior as established fact unless reinforced by repository docs, manifests, filenames, or explicit command surfaces
