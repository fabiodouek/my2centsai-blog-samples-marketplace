# Platform Detection

Before invoking the sandbox binary, detect the current platform to select the correct binary.

## Steps

1. Run `uname -s` to get the OS kernel name
2. Run `uname -m` to get the machine architecture
3. Map the values to the binary naming convention:

| `uname -s` output | Binary OS |
|---|---|
| `Darwin` | `darwin` |
| `Linux` | `linux` |
| `MINGW64_NT-*`, `MSYS_NT-*`, `CYGWIN_NT-*` | `windows` |

| `uname -m` output | Binary Arch |
|---|---|
| `arm64`, `aarch64` | `arm64` |
| `x86_64`, `amd64` | `amd64` |

4. Construct the binary name: `agentcore-sandbox-<os>-<arch>`
   - For Windows, append `.exe`: `agentcore-sandbox-windows-amd64.exe`

## Example

```bash
OS=$(uname -s | tr '[:upper:]' '[:lower:]')
ARCH=$(uname -m)

# Normalize OS
case "$OS" in
  mingw*|msys*|cygwin*) OS="windows" ;;
esac

# Normalize arch
case "$ARCH" in
  x86_64|amd64) ARCH="amd64" ;;
  arm64|aarch64) ARCH="arm64" ;;
esac

# Construct binary name
BINARY="agentcore-sandbox-${OS}-${ARCH}"
if [ "$OS" = "windows" ]; then
  BINARY="${BINARY}.exe"
fi
```

Use the resolved `$BINARY` name for all subsequent `agentcore-sandbox` invocations in this session.
