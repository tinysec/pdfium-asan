# pdfium-asan

ASAN-instrumented fuzzer build of **PDFium** — Chrome's PDF renderer. Rebuilt automatically when the revision Chrome uses changes.

![build](https://github.com/tinysec/pdfium-asan/actions/workflows/build.yml/badge.svg)
![track](https://github.com/tinysec/pdfium-asan/actions/workflows/track.yml/badge.svg)
![release](https://img.shields.io/github/v/release/tinysec/pdfium-asan?label=release)

**Engine:** libFuzzer (Linux + Windows) · **Sanitizer:** ASAN · **Symbols:** Windows `.pdb` included

`track.yml` polls Chrome stable **daily** and resolves the PDFium SHA Chrome actually ships (Chrome DEPS → `pdfium_revision`). Unchanged → nothing happens. Changed → bump + tag `chrome-<version>` → `build.yml` runs → new Release.

> Requires a `TRACK_TOKEN` PAT (`contents: write`) so the tracker's push starts the build.

```bash
git tag v1 && git push origin v1   # manual trigger
```

No harness is committed here — PDFium's built-in `pdfium_fuzzer` target (GN + depot_tools) is built from the source tree.
