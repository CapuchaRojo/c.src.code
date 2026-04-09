# Mission Read

This Phase 2 audit records a lawful, evidence-based quality-and-risk read of `c.src.code` from direct repository inspection. It follows the Phase 1 framing, samples representative files across the requested surfaces, and stays clean-room disciplined: no copied implementation logic, no patch plans, and no speculative rewrite work.

# Observed Truth

- The inspected repository root is `C:\Users\mich3\GitHubProjects\c.src.code`.
- Phase 1 correctly framed the current repo shape: root docs and CI treat `rust/` as the canonical product surface, while `src/` plus `tests/` operate as a companion Python/reference workspace.
- Root documentation, the installer, the container workflow, and GitHub Actions all converge on the Rust workspace as the operational center of gravity.
- The Rust workspace is intentionally structured as a multi-crate system with explicit lint policy at the workspace root, a dedicated CLI crate, and separate runtime, API, plugin, tool, telemetry, and harness crates.
- The repository also maintains a large Python-side mirror/reference surface with inventory, manifest, routing, and parity-audit utilities backed by snapshot data.
- Validation posture is mixed but real: CI enforces documentation source-of-truth, formatting, tests, and clippy for Rust; the Rust crates include integration-heavy tests; the Python/reference layer has one broad end-to-end test module.
- The repo contains tracked local-looking runtime artifacts under `rust/.claude/`, `rust/.claw/`, and `rust/.sandbox-home/`, while the root `.gitignore` only ignores top-level local session paths.

# Strengths

- The repository has a clear primary runtime boundary. `README.md`, `USAGE.md`, `install.sh`, `docs/container.md`, and `.github/workflows/rust-ci.yml` all point contributors toward `rust/` instead of leaving the active product surface ambiguous.
- Quality controls are visible and practical. The Rust CI workflow checks formatting, runs the workspace tests, runs clippy, and includes a dedicated documentation-branding guard script to catch stale source-of-truth drift.
- The Rust workspace shows deliberate modularity. The crate split separates CLI, runtime, API, tools, plugins, telemetry, and harness concerns rather than collapsing everything into one binary crate.
- Safety posture is not cosmetic. The workspace lint configuration forbids `unsafe_code`, the runtime surface is organized around permissions and sandboxing concepts, and the parity/test materials emphasize permission-denied and boundary scenarios instead of only happy paths.
- The test strategy includes behavior-level harnessing, not just unit assertions. The mock parity harness and integration-oriented Rust tests indicate that the project is trying to validate end-to-end flows across tool, permission, plugin, and output boundaries.

# Weaknesses

- The repository carries two substantial architectural narratives at once. Even though the docs state that `rust/` is canonical, the size and breadth of `src/` plus `tests/` still create maintenance and interpretation overhead.
- Documentation and status surfaces are numerous and active. `README.md`, `USAGE.md`, `PARITY.md`, `ROADMAP.md`, `PHILOSOPHY.md`, `CLAUDE.md`, and Rust-local docs all need to stay aligned, which raises drift risk even with the branding checker in place.
- Some modules and test surfaces are very large, especially in plugin and parity areas. That is not automatically wrong, but it raises review difficulty and increases the chance that unrelated concerns accrete in one place.
- The Python/reference test posture is concentrated in a single broad test module. That gives useful smoke coverage, but it makes the reference layer look more monolithic than segmented.
- Portability messaging is narrower than the repository surface might first suggest. The installer explicitly redirects Windows users to WSL, and the release workflow only builds a limited binary matrix.

# Risks / Watchouts

- The largest structural risk is source-of-truth confusion over time. Phase 1 already identified the split between canonical Rust runtime and companion Python/reference workspace; Phase 2 inspection confirms that both layers are still substantial enough to confuse future maintainers if documentation discipline slips.
- Tracked local/session-style artifacts under `rust/` are a maintenance and hygiene watchout. Their presence makes it easier for environment-specific state to look authoritative simply because it is versioned.
- The parity/status narrative needs careful trust calibration. `PARITY.md` is rich and specific, but it also mixes milestone accounting, behavioral claims, and open-status statements, so readers could over-treat it as a direct runtime guarantee if they do not cross-check code and CI.
- Plugin and hook extensibility is powerful but increases risk surface. The repo clearly treats plugins, hooks, and external tool execution as first-class seams, which means reliability and boundary clarity matter more than in a closed CLI.
- The repository's docs showed visible character-rendering noise during terminal inspection in multiple markdown files. Even if partly terminal-related, it is still a portability/documentation watchout because encoding friction degrades operator confidence.

# Safe Lessons for clau-dex

- Keep one clearly named canonical runtime surface, and reinforce it in docs, CI, installer flow, and container guidance the way this repo does for `rust/`.
- Pair architectural modularity with visible operational checks. The strongest pattern here is not just crate separation, but crate separation backed by CI, source-of-truth checks, and behavior-oriented tests.
- Treat permissions, sandboxing, and boundary-denial scenarios as quality work, not optional hardening. This repo repeatedly validates denial paths and workspace boundaries, which is a safe lesson worth carrying forward.
- When a mirror or reference workspace exists, label it aggressively as secondary and keep its purpose narrow. The repo's current docs help, but the size of the companion layer shows why this boundary must stay explicit.
- Preserve clean-room reporting discipline when auditing adjacent systems: grounded observations, representative evidence, and lessons-only framing are more portable than implementation transplant notes.

# Do-Not-Import List

- Do not import dual-runtime ambiguity where two large trees both look like the real product surface.
- Do not import tracked session, sandbox, or tool-state artifacts into the long-term source tree unless they are intentionally governed reference fixtures.
- Do not import milestone/status documents as substitutes for executable verification.
- Do not import extension seams without equally visible validation and boundary-language around permissions, hooks, and external execution.
- Do not import giant mixed-responsibility files merely because they already exist in a mature repo; they increase audit cost and obscure ownership lines.

# Open Questions

- Which checked-in local-style artifacts under `rust/` are intentional fixtures versus incidental workspace residue is not fully explained by the inspected docs.
- The long-term governance contract for the Python/reference workspace is still only partially explicit. Its role is described, but its desired stability and retirement horizon are not consolidated in one obvious place.
- The release workflow's platform scope appears intentionally limited, but the repo does not prominently explain whether that is a temporary checkpoint decision or the expected steady-state distribution model.
- The repo-visible name `c.src.code` and the product branding `Claw Code` coexist without a short explanation in the inspected docs.

# Clean-Room Notes

- This report is based on direct inspection of representative repository files, including `docs/ARCHITECTURE_AUDIT_PHASE1.md`, root docs and operational files, `.github/`, `rust/`, `src/`, and `tests/`.
- It intentionally avoids copied code, close paraphrase of implementation bodies, patch plans, refactor plans, and feature proposals.
- It focuses on quality signals, risks, safe lessons, and non-import warnings rather than implementation design transfer.

# Handoff Summary

The Phase 2 read supports Phase 1's central framing: this repository has a strong canonical Rust runtime with real quality scaffolding, but it also carries a sizable companion reference layer and a broad documentation/status surface that increase governance load. Its strongest qualities are boundary awareness, visible CI discipline, modular Rust organization, and behavior-oriented validation. Its main watchouts are source-of-truth drift, tracked local-style artifacts, extension-surface complexity, and the long-term maintenance cost of keeping the Python/reference mirror substantial.
