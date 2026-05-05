ARCHITECTURE — stuxnet (owner-provided snapshot)

WARNING: This document is a high-level architectural analysis intended for defensive, research, and archival purposes only. It intentionally omits code-level reproduction details, exploit steps, shellcode, and precise kernel/user API usage that could enable unsafe re-use.

1. Executive summary

The codebase contains two primary functional families:

- User-mode staging/loader components (in `dropper/`): responsible for carrying encoded/obfuscated artifacts in repository sources, performing decoding/rehydration, and handing artifacts to the installation path.
- Kernel-mode interception components (in `rootkit/`): reconstructed kernel driver code intended to integrate with the OS at a low level, intercepting or modifying kernel I/O paths to provide stealth and persistence for user-mode artifacts.

These two families form a pipeline: a user-mode component stages and installs payloads; a kernel-mode component provides concealment and long-term persistence for those payloads.

2. Major components and responsibilities (conceptual)

- Stager/Dropper (user-mode)
  - Purpose: carry embedded or encoded payload material inside benign-looking containers, decode/translate the data into a runnable form, and place it where it can be (optionally) loaded.
  - Typical responsibilities: data decoding, integrity checks, writing files to disk (staging), invoking or handing off an installation/loading sequence.
  - Interfaces: file system for staging, (optionally) an installation loader that triggers kernel component installation.

- Kernel component / Rootkit (kernel-mode)
  - Purpose: intercept specific kernel I/O or driver paths to hide files/objects, alter system view, and maintain persistence across reboots.
  - Typical responsibilities: installing hooks/interception points in kernel subsystems, filtering or modifying enumeration results, and responding to control inputs from user-mode components.
  - Interfaces: kernel driver interface and system kernel subsystems (device/driver layers) to effect concealment.

- Assembly/obfuscation fragments
  - Purpose: implement tight loops, obfuscation, or low-level stubs that are difficult to express in C alone. Found in assembly blocks and encoded constants.
  - Role: make static analysis harder by scattering logic in low-level sequences and encoded tables.

3. Data and control flow (high-level)

- Source materials: repository contains encoded constants and assembly fragments rather than plain payload binaries.
- Decode path: a sequence in the user-mode component reads encoded constants, transforms them, and writes out artifact(s) to staging paths.
- Load/hand-off: once staged, an installation sequence (user-mode initiated) places the artifact where the kernel component or loader will pick it up, or directly triggers activation.
- Kernel activation: kernel component, once present, alters system behavior (enumeration, I/O) to hide or protect staged artifacts and possibly accept further commands from user-mode.

4. Architectural separations and coupling

- Separation of concerns: user-mode handles staging/transport/decoding; kernel-mode handles concealment and persistence.
- Coupling points: file system paths, installation sequence (signed/unpacked driver or kernel object), and a control channel between user-mode and kernel-mode components.
- Persistence: accomplished by placing artifacts in locations that the kernel component conceals, plus installing reactive kernel logic to survive restarts.

5. Defensive indicators (non-actionable)

The following are defensive signals for triage and detection (high-level patterns rather than reproduction-ready artifacts):

- Presence of encoded/obfuscated constant tables or large hex arrays in source files.
- Atomic blocks of handwritten assembly in otherwise-C projects.
- Source comments or filenames indicating reconstructed/"reversed" code from known samples.
- Kernel-driver source that references driver/fast-io/file-enumeration interception concepts.

Suggested detection approaches:
- Static scanning for large encoded blobs and assembly fragments.
- File provenance tagging (label samples as forensic/archival) and restricting execution rights.
- Use of YARA-like rules to detect the artifact patterns (owner-controlled and validated rules only).

6. Safety and handling recommendations

- Do not compile or execute any components outside an isolated, air-gapped analysis environment managed by security professionals.
- Keep a separate archival snapshot of the unmodified repository for forensic tracking.
- If sharing artifacts for teaching, produce a sanitized package that contains only documentation and stripped analysis notes; keep raw artifacts behind access control.

7. Repository hygiene and documentation suggestions

- Add an explicit top-level `NOTICE.md` stating: "This repository contains reconstructed malware artefacts. Do not compile or run. For research, contact owner and follow institutional procedures."
- Add `CONTRIBUTING.md` and `LICENSE` clarifying permitted uses and distribution restrictions.
- Consider splitting the repo into: `analysis/` (notes, docs) and `samples/` (raw reconstructed artifacts) with `samples/` archived under strict access control.

8. Offer: file-by-file annotated report

I can produce a detailed, file-by-file annotated report that summarizes each source file's role and notes for defensive analysts. I will NOT include disallowed, actionable content (assembly reproduction, precise loader/driver commands, or sample payloads). Reply if you want that report and whether it should be added to the repository as `FILE_NOTES.md` or provided here inline.

---

If you want the file-by-file report, confirm and I will produce it next (one file summary per entry).