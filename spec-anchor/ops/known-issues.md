# Known Issues & Lessons Learned — Sent2_GreenMac

> Last updated: 2026-04-14

## Active Issues

_None._

## Resolved Issues

### rasterio / osgeo.gdal fail with `_locale_charset` not found (macOS ARM64, cdsetool)

- **Symptom**: `ImportError: Symbol not found: _locale_charset` when importing rasterio or
  `from osgeo import gdal` in the cdsetool conda environment on Apple Silicon.
- **Root Cause**: `libspatialite.8.dylib` (pulled in by rasterio and gdal Python wheels)
  calls `_locale_charset`.  Our `$JRE/lib/libiconv.2.dylib` wrapper (which DYLD_LIBRARY_PATH
  places ahead of conda's libiconv) did not export this symbol.
- **Fix**: Recompile the wrapper to also export `locale_charset`, delegating to conda's
  libiconv (which contains `_locale_charset`).  See `docs/install-apple-silicon.md` step 2b
  in the SNAP repo for the full wrapper source.  After adding the symbol, verify with
  `nm "$JRE/lib/libiconv.2.dylib" | grep locale_charset`.
- **Status**: Resolved 2026-04-14.

---

## Lessons Learned

### DYLD_LIBRARY_PATH wrapper scope

When a dylib wrapper is placed on DYLD_LIBRARY_PATH to shadow a conda library, it must
export *all* symbols that any downstream library expects from that filename — not just the
ones originally needed.  Discovering a new consumer (libspatialite) required a wrapper
recompile.  Keep the wrapper source in version control and document every symbol it exports.
