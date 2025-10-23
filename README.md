# Python 3.14t for Termux (Free-Threading)

[![Python Version](https://img.shields.io/badge/python-3.14.0-blue.svg)](https://www.python.org/downloads/release/python-3140/)
[![Architecture](https://img.shields.io/badge/arch-ARM64%20(aarch64)-orange.svg)](https://developer.android.com/ndk)
[![License](https://img.shields.io/badge/license-PSF-green.svg)](https://docs.python.org/3/license.html)

**Python 3.14.0** with **free-threading** (GIL disabled) compiled for **Termux on Android ARM64**.

🎯 True parallel multi-threading on Android!
⚡ 1.5-2.5x speedup for CPU-bound tasks on multi-core devices
📱 Ready for Termux - no root required

---

## What is this?

This is a pre-built Python 3.14.0 interpreter with **free-threading** enabled (`--disable-gil`), specifically compiled for Termux on Android ARM64 devices.

**Free-threading** (PEP 703) removes the Global Interpreter Lock (GIL), allowing true parallel execution of Python code across multiple CPU cores.

### Why use this?

- 🚀 **Better performance**: CPU-bound multi-threaded code runs 1.5-2.5x faster
- 🔬 **Experiment**: Test the latest Python 3.14 features on Android
- 🛠️ **Development**: Build apps that leverage multi-core mobile processors
- 📚 **Learning**: Understand free-threading without building from source

---

## Requirements

- **Device**: Android 7.0+ (API 24+)
- **Architecture**: ARM64 (aarch64) - check with `uname -m`
- **Termux**: Latest version from [F-Droid](https://f-droid.org/packages/com.termux/)
- **Storage**: ~100MB free space

---

## Quick Installation

### Step 1: Download the package

```bash
# In Termux, download the latest release
wget https://github.com/fibogacci/python314t-for-termux/releases/download/v3.14.0/termux-python314t-3.14.0.tar.gz
```

### Step 2: Extract and install

```bash
# Extract the archive
tar -xzf termux-python314t-3.14.0.tar.gz

# Navigate to the directory
cd termux-python314t-3.14.0

# Run the installer
bash install.sh
```

The installer will:
- Copy files to `$HOME/.local/` (or custom `PREFIX`)
- Create symlinks: `python3.14t`, `python3`, `python`
- Set up the Python 3.14t environment

### Step 3: Configure environment

Add to your `~/.bashrc`:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH="$HOME/.local/lib:$LD_LIBRARY_PATH"' >> ~/.bashrc
source ~/.bashrc
```

### Step 4: Verify installation

```bash
# Check version
python3.14t --version

# Expected output: Python 3.14.0 free-threading build ...
```

```bash
# Verify GIL is disabled
python3.14t -c "import sys; print(f'GIL enabled: {sys._is_gil_enabled()}')"

# Expected output: GIL enabled: False
```

✅ If you see `False`, free-threading is working!

---

## Package Management - Use uv (Required)

**⚠️ Important:** Standard `pip` does **NOT work** with this free-threading build due to ctypes library naming issues (`libpython3.14t.so` vs `libpython3.14.so`).

**Use `uv` instead** - a fast Python package installer that works perfectly with free-threading Python:

### Install uv

```bash
# Install from Termux repository
pkg install uv
```

### Create a project

```bash
# Initialize new project with Python 3.14t
uv init my-project --python 3.14t
cd my-project

# Add packages
uv add requests numpy

# Run Python
uv run python --version
# Output: Python 3.14.0 free-threading build

# Run your scripts
uv run python script.py
```

### Why uv?

- ✅ Works with free-threading Python 3.14t (pip doesn't!)
- ⚡ 10-100x faster than pip
- 🔒 Reliable dependency resolution
- 📦 Automatic virtual environment management
- 🐍 Compatible with all Python packages

Learn more: https://github.com/astral-sh/uv

---

## Testing Free-Threading Performance

Create a benchmark script to test multi-threading:

```bash
cat > benchmark.py << 'EOF'
import sys
import time
from concurrent.futures import ThreadPoolExecutor

def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

print(f"Python: {sys.version}")
print(f"GIL enabled: {sys._is_gil_enabled()}")
print()

# Single-threaded
n = 35
start = time.time()
result = fibonacci(n)
single_time = time.time() - start
print(f"Single-threaded Fib({n}): {single_time:.2f}s")

# Multi-threaded (4 workers)
start = time.time()
with ThreadPoolExecutor(max_workers=4) as executor:
    futures = [executor.submit(fibonacci, n) for _ in range(4)]
    results = [f.result() for f in futures]
multi_time = time.time() - start

print(f"Multi-threaded (4x Fib({n})): {multi_time:.2f}s")
print(f"Speedup: {(single_time * 4) / multi_time:.2f}x")

if sys._is_gil_enabled():
    print("⚠️  GIL enabled - limited parallelism")
else:
    print("✅ Free-threading active - real parallelism!")
EOF

python3.14t benchmark.py
```

**Expected results** (4-core ARM device):
- Speedup: **1.5-2.5x**
- Output: `✅ Free-threading active - real parallelism!`

---

## Troubleshooting

### Library not found error

```bash
# Error: libpython3.14t.so: cannot open shared object file

# Solution: Add library path
export LD_LIBRARY_PATH="$HOME/.local/lib:$LD_LIBRARY_PATH"

# Make permanent
echo 'export LD_LIBRARY_PATH="$HOME/.local/lib:$LD_LIBRARY_PATH"' >> ~/.bashrc
```

### Wrong architecture

```bash
# Error: cannot execute binary file: Exec format error

# Check your architecture
uname -m

# This package requires: aarch64 (ARM64)
# If you have armv7l (32-bit), this package won't work
```

### Python not found

```bash
# Error: python3.14t: command not found

# Add to PATH
export PATH="$HOME/.local/bin:$PATH"

# Make permanent
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### Import errors (missing modules)

Some standard library modules were removed to reduce size:
- `test/` - test suite (use `pytest` instead)
- `idlelib/` - IDLE IDE (not useful on Termux)
- `tkinter/` - Tk/Tcl GUI (not available on Android)

All other standard modules are included.

---

## Package Contents

- **Binary**: `python3.14t` (ARM aarch64 executable)
- **Library**: `libpython3.14t.so` (31MB shared library)
- **Standard library**: `python3.14t/` (34MB, cleaned)
- **Headers**: `include/python3.14t/` (C headers for extensions)
- **Scripts**: `install.sh`, `README.md`

**Total size**: 21MB compressed, 67MB uncompressed

---

## Building from Source

If you want to build Python 3.14t yourself:

```bash
# Install dependencies
pkg install build-essential clang openssl libffi zlib sqlite ncurses readline

# Download Python source
wget https://www.python.org/ftp/python/3.14.0/Python-3.14.0.tar.xz
tar -xf Python-3.14.0.tar.xz
cd Python-3.14.0

# Configure with free-threading
./configure --prefix=$HOME/.local --disable-gil --enable-shared

# Build (takes 1-3 hours on mobile!)
make -j4

# Install
make install
```

**Note**: Cross-compilation from a Linux PC is much faster (~17 minutes). See [build documentation](docs/BUILD.md) for details.

---

## Compatibility

### Known working packages

✅ **Tested and working:**
- `requests`, `urllib3` - HTTP libraries
- `numpy` - Numerical computing (with compatible wheels)
- `asyncio` - Async I/O (built-in)
- `threading` - Multi-threading (built-in)
- `concurrent.futures` - Thread/process pools

⚠️ **May require recompilation:**
- C extension modules built for standard Python (with GIL)
- Check [free-threading compatibility tracker](https://py-free-threading.github.io/)

### Performance expectations

**CPU-bound multi-threaded**: 1.5-2.5x speedup (4 cores)
**I/O-bound**: Similar to standard Python
**Single-threaded**: ~5-10% slower (free-threading overhead)

**Best for**: Parallel processing, concurrent tasks, multi-user servers

---

## Resources

- [Python 3.14 Release](https://www.python.org/downloads/release/python-3140/)
- [PEP 703: Free-Threading](https://peps.python.org/pep-0703/)
- [Free-Threading Guide](https://docs.python.org/3.14/howto/free-threading-python.html)
- [Termux Wiki](https://wiki.termux.com/)
- [uv Package Manager](https://github.com/astral-sh/uv)

---

## License

Python 3.14 is licensed under the [Python Software Foundation License](https://docs.python.org/3/license.html).

This packaging and distribution is provided as-is for the Termux community.

---

## Credits

Built using the official [CPython Android build system](https://github.com/python/cpython/tree/main/Android).

Free-threading enabled via `--disable-gil` flag (PEP 703).

Cross-compiled for Termux ARM64 with optimization and size reduction.

---

## Contributing

Issues, suggestions, and contributions welcome!

- Report bugs: [GitHub Issues](https://github.com/fibogacci/python314t-for-termux/issues)
- Discussions: [GitHub Discussions](https://github.com/fibogacci/python314t-for-termux/discussions)

---

**Enjoy free-threading Python on Android!** 🚀🐍📱
