# pdfium-asan

ASAN-instrumented build of **PDFium** — Chrome's PDF renderer. Rebuilt automatically whenever the revision Chrome uses changes.

![build](https://github.com/tinysec/pdfium-asan/actions/workflows/build.yml/badge.svg)
![track](https://github.com/tinysec/pdfium-asan/actions/workflows/track.yml/badge.svg)
![release](https://img.shields.io/github/v/release/tinysec/pdfium-asan?label=release)

## Staying current

`chrome.lock` pins the exact PDFium revision a specific Chrome stable version
ships (PDFium is pinned directly in Chrome DEPS). The
[`track-chrome`](.github/workflows/track.yml) workflow runs every 6 hours: it
resolves the latest Chrome stable, reads the PDFium SHA from Chrome DEPS, and
when it has changed it bumps `chrome.lock`, tags `pdfium-<sha8>`, and
triggers [`build`](.github/workflows/build.yml). Each `pdfium-<sha8>` tag
becomes a GitHub release.

## Release artifacts

Each release is published at its `pdfium-<sha8>` tag as **one zip per
platform** (so both legs' `libpdfium.a` and the shared `public/*.h` headers do
not collide in GitHub's flat asset namespace). Each zip contains:

- `lib/libpdfium.a`: the ASAN **static** library — every compiled PDFium
  object (excluding the `testing/` tree) archived into one static lib. Link it
  into a harness that calls the PDFium API. This is the primary artifact for
  fuzzing. (PDFium is shipped static only; no dynamic ASAN lib.)
- `lib/include/`: the 24 PDFium public headers (`fpdfview.h` is the main entry
  point; plus `fpdf_annot.h`, `fpdf_edit.h`, `fpdf_text.h`, …).
- `fuzz/pdfium_test` (Linux) / `fuzz/pdfium_test.exe` (Windows): the
  standalone, own-`main()` ASAN PDF renderer — a ready-to-fuzz target with no
  harness needed (WinAFL drives it directly on its file-input entry point).

## ASAN runtime model

Both platforms link the **static** ASAN runtime (depot_tools' bundled clang
ships the full static sanitizer runtime). The library and `pdfium_test` are
self-contained — no `clang_rt.asan_dynamic` DLL is needed at runtime.

## Fuzzing

- **Windows (AFL++ / WinAFL):** the simplest path is to point WinAFL directly
  at `pdfium_test.exe`'s file-input entry point — no harness to write.
  Alternatively, link `lib/libpdfium.a` into a custom harness.
- **Linux:** `pdfium_test` is an ASAN executable with its own `main()` (replays
  a PDF file).

> PDFium standalone ships no libFuzzer-engine binary by design (the upstream
> `testing/fuzzers/BUILD.gn` deliberately does not create fuzzer executables —
> that is done in Chromium). These builds provide the ASAN-instrumented library
> + `pdfium_test` to target; they do not bundle the AFL++ / WinAFL runner
> itself (a separate DynamoRIO-based toolchain on Windows).
