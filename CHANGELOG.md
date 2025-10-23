# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [3.14.0] - 2025-10-23

### Added
- Initial release of Python 3.14.0 for Termux ARM64
- Free-threading build (GIL disabled via `--disable-gil`)
- Pre-built package: 21MB compressed, 67MB uncompressed
- Automated installer script (`install.sh`)
- README with installation and usage instructions
- Benchmark example for testing multi-threading performance

### Fixed (2025-10-24)
- **install.sh:** Auto-configure ~/.bashrc with interactive prompt
- **install.sh:** Force PREFIX to ~/.local to avoid system conflicts
- **install.sh:** Only install python3.14t binary (no symlink overwrites)
- **install.sh:** Check if PATH/LD_LIBRARY_PATH already configured
- **README.md:** Simplified to essential installation steps
- **README.md:** Added source ~/.bashrc reminder
- **README.md:** Clearer verification instructions

### Technical Details
- Architecture: ARM aarch64 (64-bit)
- Platform: Android 7.0+ (API 24+)
- Build method: Cross-compilation from Linux x86_64
- Build time: ~17 minutes
- Compiler: Clang 18.0.4 (Android NDK 27.3.13750724)

### Package Contents
- `python3.14t` executable (6.9KB)
- `libpython3.14t.so` shared library (31MB)
- Python 3.14t standard library (34MB, optimized)
- C headers for extension development (2.6MB)

### Performance
- Multi-threaded CPU-bound tasks: 1.5-2.5x speedup on 4-core devices
- Single-threaded overhead: ~5-10% (expected for free-threading)

### Known Limitations
- pip not included by default (install via `get-pip.py`)
- Some C extensions may need recompilation for free-threading
- GUI modules removed (`tkinter`, `idlelib`)
- Test suite removed for size optimization

### Testing
- Tested on real Termux Android ARM64 device
- Verified installation process
- Confirmed free-threading works (GIL disabled)
- Performance benchmarks validated

---

[3.14.0]: https://github.com/fibogacci/python314t-for-termux/releases/tag/v3.14.0
