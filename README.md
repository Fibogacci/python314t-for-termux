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
echo 'export SSL_CERT_FILE="/data/data/com.termux/files/usr/etc/tls/cert.pem"' >> ~/.bashrc
source ~/.bashrc
```

**Note:** Make sure `ca-certificates` package is installed:
```bash
pkg install ca-certificates
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

## Verify SSL Support

Python 3.14t includes **OpenSSL 3.3.2** for secure HTTPS connections:

```bash
python3.14t -c "import ssl; print(ssl.OPENSSL_VERSION)"
# Output: OpenSSL 3.3.2 3 Sep 2024

# Test HTTPS connection
python3.14t -c "import urllib.request; print(urllib.request.urlopen('https://www.python.org').status)"
# Output: 200
```

**Note:** SSL certificates are configured to use Termux CA bundle. Ensure `ca-certificates` is installed:
```bash
pkg install ca-certificates
```

✅ SSL/TLS fully supported!

---

## Package Management - Use pip

### Install pip

```bash
# Download pip installer
wget https://bootstrap.pypa.io/get-pip.py

# Install pip for Python 3.14t
python3.14t get-pip.py

# Verify installation
pip --version
```

### Install packages

```bash
# Install pure Python packages
pip install requests httpx click

# Test requests
python3.14t -c "import requests; print(requests.get('https://httpbin.org/get').json())"
```

### Why not uv?

**⚠️ Important:** `uv` package manager does **NOT work** on Android/Termux:

```bash
$ uv add requests
error: Can't use Python at `/data/data/com.termux/files/home/.local/bin/python3.14t`
Caused by: Unknown operating system: `android`
```

uv does not recognize Android as a supported platform. **Use pip instead.**

---

## Package Compatibility

✅ **Pure Python packages work perfectly:**
- `requests`, `httpx`, `urllib3` - HTTP libraries (HTTPS supported!)
- `click`, `typer` - CLI frameworks
- `pydantic` - Data validation
- `beautifulsoup4` - HTML/XML parsing
- `asyncio` - Async I/O (built-in)
- `threading`, `concurrent.futures` - Multi-threading (built-in)

⚠️ **C extension packages (numpy, pandas, scipy) may NOT work:**
- Termux uses Android **Bionic libc** (not glibc)
- PyPI wheels are built for standard Linux (**manylinux** - glibc-based)
- Platform mismatch: `manylinux_aarch64` ≠ `android_aarch64`
- These packages require building from source with Android NDK

**Recommendation:** Stick to **pure Python packages** for best compatibility and reliability.

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

### Package compatibility

✅ **Pure Python packages work perfectly:**
- `requests`, `httpx`, `urllib3` - HTTP libraries
- `click`, `typer` - CLI frameworks
- `pydantic` - Data validation
- `beautifulsoup4` - HTML parsing
- `asyncio` - Async I/O (built-in)
- `threading`, `concurrent.futures` - Multi-threading (built-in)

⚠️ **C extensions may NOT work (Bionic vs glibc):**
- `numpy`, `pandas`, `scipy` - Scientific computing
- Most PyPI wheels are for `manylinux` (glibc), not Android (Bionic)
- Require building from source with Android NDK toolchain

### Performance expectations

**CPU-bound multi-threaded**: 1.5-2.5x speedup (4 cores)
**I/O-bound**: Similar to standard Python
**Single-threaded**: ~5-10% slower (free-threading overhead)

**Best for**: Scripting, automation, web scraping, CLI tools, async I/O, HTTPS requests

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

## Author

**Fibogacci**

- Website: [Fibogacci.com](https://fibogacci.com)
- Blog: [AndroidPython.com](https://androidpython.com) - Python on Android tutorials and resources

---

**Enjoy free-threading Python on Android!** 🚀🐍📱
