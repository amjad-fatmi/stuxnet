FILE NOTES — per-file, non-actionable summaries

WARNING: This repository contains reconstructed artifacts from malware binaries. The notes below intentionally avoid code excerpts, reproduction steps, exact assembly or API calls, and any details that could enable misuse. Use these notes for defensive triage, documentation, and archival purposes only.

Top-level

- `README.md` — Owner-provided context indicating historical provenance and that parts were extracted from malware binaries. Treat as forensic metadata; consider updating to clarify permitted uses.
- `ARCHITECTURE.md` — High-level architectural analysis (added by reviewer). Provides conceptual overview and safety guidance.
- `CODE_OVERVIEW.md` — Short summary describing the repository layout and major modules.

Directory: `dropper/`

- `1. Main.c` — Entry-level user-mode control logic for staging/installation. Orchestrates higher-level sequences (decoding, staging, hand-off). Sensitive: contains sequencing logic that, if executed, could recreate staging behavior.

- `2. STUBHandler.c` / `2. STUBHandler.h` — Abstraction layer for calling into low-level stubs and managing platform differences. Contains dispatching logic and interfaces used by assembly fragments.

- `3. OS.c` / `3. OS.h` — Platform abstraction helpers (filesystem and environment interactions). Useful for understanding staging targets and environmental assumptions.

- `4. Encoding.c` / `4. Encoding.h` — Encoding/obfuscation tables and decoding orchestration. Implements transformations over embedded data blobs. Highly sensitive: these files describe how embedded data is transformed into payload material.

- `5. Utils.c` / `5. Utils.h` — Utility routines used across the dropper code (parsing, byte operations, logging-like helpers). Low-level helpers that support decoding and staging paths.

- `6. MemorySections.c` / `6. MemorySections.h` — Helpers for handling in-memory sections, layouts, and copying staged artifacts into memory structures. Relevant to how the dropper maps decoded data for later use.

- `7. AssemblyBlock0.c` / `7. AssemblyBlock0.h` — Assembly fragments embedded in C sources. Contain handwritten low-level sequences that are intentionally terse and obfuscated.

- `8. AssemblyBlock1.c` / `8. AssemblyBlock1.h` — Additional low-level assembly fragments. These are implementation artifacts that complicate static analysis and often perform tight transformations.

- `9. AssemblyBlock2.c` / `9. AssemblyBlock2.h` — Further assembly sequences; includes comparisons and low-level control flow markers. Treat as sensitive and avoid reproducing.

- `A. EncodingAlgorithms.c` / `A. EncodingAlgorithms.h` — Encapsulation of encoding/decoding algorithms invoked by `Encoding.c`. Implements algorithmic kernels (non-actionable description only).

- `C. CodeBlock.c` / `C. CodeBlock.h` — Contain code block management routines, glue between decoded blobs and staging/writing logic.

- `config.h`, `define.h`, `StdAfx.h` — Build and configuration headers; define compile-time constants and environmental assumptions.

- `export.def` — Export definitions for linking. Useful for build reproduction but also indicates exported symbols that may be used by loaders.

- `LICENSE` — License file present (review for intended distribution). Confirm if the repo owner wants it retained or changed.

Directory: `rootkit/`

- `main.c` — Kernel-mode driver reconstruction entry points (per source comment). Contains driver initialization and registration scaffolding. Highly sensitive: kernel-mode logic modifies OS-level behavior.

- `FastIo.c` — Fast I/O interception routines reconstructed from MRxNet rootkit artifacts (per comments). Implements interception/fast-path hooks that alter file/device enumeration or I/O results.

- `LICENSE` — License file for the rootkit module (review scope as above).

General notes and triage guidance

- Sensitivity: The most sensitive files are the encoding/decoding modules, assembly blocks, and kernel-mode sources. All contain functionality that can materially change system state if compiled and executed.

- Forensic use: If you intend others to examine this repository, add clear access controls and a top-level `NOTICE` or `HANDLING.md` that specifies: "Do not compile or run. For accredited research contact owner."

- Static triage suggestions (non-actionable):
  - Produce per-file metadata comments describing provenance (which binary sample, extraction date, limitations of reconstruction).
  - Tag files that are partial reconstructions with a header note: "Partial reconstruction — may be incomplete or altered."
  - Add a `FILE_NOTES.md` (this file) and include safe pointers for defensive analysts (e.g., YARA patterns for detection — owner-supplied only).

- Next steps I can take (owner choice):
  - Produce a sanitized teaching copy that removes or redacts sensitive implementation files (I previously created one but removed it at owner request). I will only proceed after explicit owner confirmation.
  - Produce a deeper static analysis report that highlights suspicious strings, exported symbols, and structural call graph notes — the report will not include exploit, shellcode, or step-by-step payload reconstruction.

If you want the deeper static analysis report, confirm and I'll produce it. If you want changes committed to the repo (documentation-only), tell me which files to add or update and I'll prepare a branch and show diffs before committing.