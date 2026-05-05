DETAILED STATIC REPORT — stuxnet (owner-provided snapshot)

WARNING: This report is intentionally non-actionable. It summarizes static artifacts (symbols, headers, array declarations, and indicators) to support defensive analysis and documentation. It omits code excerpts, assembly listings, decoding logic, and any step-by-step instructions that could enable misuse.

Scan date: 2026-05-04
Scanned path: /Users/xquark_home/stuxnet

1) Export table (dropper/export.def)
- Present and lists exported entry points (DLL-style exports):
  - CPlApplet
  - DllCanUnloadNow
  - DllRegisterServer (mapped to DllGetClassObject)
  - DllUnregisterServer (mapped to DllRegisterServerEx)
  - DllRegisterServerEx
  - DllUnregisterServerEx
  - DllGetClassObject
  - DllGetClassObjectEx

Implication: the dropper sources include a module built with exported symbols consistent with a DLL-style component.

2) Function headers (per-source scan)
- A project-level function index was attempted with `ctags` (if available) or a conservative grep fallback; an environment CTags variant reported incompatible flags on this system, so rely on the conservative extraction approach.
- Files scanned: all `dropper/*.c` and `rootkit/*.c`.
- Observed high-level entry/coordination files: `dropper/1. Main.c`, `dropper/3. OS.c`, `dropper/6. MemorySections.c`, `rootkit/main.c`.
- Notes: function-level names and prototypes are present in source; these are best reviewed within the repository under controlled conditions.

3) Large constant / array declarations (defensive index)
The scan located named encoded constant tables (non-executable listing omitted). These are indicators of obfuscated API names or encoded payload material.
- `dropper/4. Encoding.c` contains multiple `const WORD ENCODED_*` tables, including (names only):
  - `ENCODED_lstrcmpiW`
  - `ENCODED_VirtualQuery`
  - `ENCODED_VirtualProtect`
  - `ENCODED_GetProcAddress`
  - `ENCODED_MapViewOfFile`
  - `ENCODED_UnmapViewOfFile`
  - `ENCODED_FlushInstructionCache`
  - `ENCODED_LoadLibraryW`
  - `ENCODED_FreeLibrary`
  - `ENCODED_ZwCreateSection`
  - `ENCODED_ZwMapViewOfSection`
  - `ENCODED_CreateThread`
  - `ENCODED_WaitForSingleObject`
  - `ENCODED_GetExitCodeThread`
  - `ENCODED_ZwClose`
  - `ENCODED_CreateRemoteThread`
  - `ENCODED_NtCreateThreadEx`
- `dropper/5. Utils.c` contains `ENCODED_KERNEL32_DLL_ASLR__08x` (name only).
- `dropper/A. EncodingAlgorithms.c` contains `ENCODED_NTDLL_DLL` and `ENCODED_KERNEL32_DLL`.

Implication: source includes many encoded symbol/strings tables; these are high-sensitivity artifacts for defensive analysts.

4) Assembly / low-level fragments
- Files referencing or including assembly headers: `dropper/7. AssemblyBlock0.c`, `dropper/8. AssemblyBlock1.c`, `dropper/9. AssemblyBlock2.c`, and several other C files include the corresponding headers.
- These assembly blocks are used across `MemorySections.c`, `Utils.c`, and `CodeBlock.c`.

Implication: low-level handwritten sequences are present and intentionally obfuscatory; avoid reproducing these fragments outside air-gapped analysis.

5) Suspicious keywords / provenance markers
- Files that contain explicit provenance or suspicious markers (found via keyword scan):
  - `dropper/4. Encoding.c` — contains at least one provenance/suspicious keyword occurrence.
  - `rootkit/FastIo.c` — contains at least one provenance/suspicious keyword occurrence.
  - `rootkit/main.c` — contains at least one provenance/suspicious keyword occurrence.

Typical keywords matched: `revers` (part of "reversed"), `extracted`, `Stuxnet`, `rootkit`, `MRxNet`, plus other defensive indicators.

