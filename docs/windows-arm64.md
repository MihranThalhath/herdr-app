# Native Windows ARM64 build

herdr-app builds natively for `aarch64-pc-windows-msvc` (PE `0xAA64`) on Windows
ARM64 hardware. Upstream release manifests only ship `windows-x86_64`.

## Toolchain

| Piece | Notes |
|---|---|
| Rust | repo-pinned toolchain (see `rust-toolchain.toml` / workspace pin) |
| MSVC | VS with `Hostarm64\arm64\link.exe` |
| Windows SDK | 10.0.26100+ (needs `arm64\fxc.exe` for shaders) |
| Zig 0.16.0 | **x86_64 build** — native ARM64 Zig crashes; x86_64 runs under emulation |
| LLVM/libclang | aarch64 Windows build; `libclang.dll` for bindgen, MSVC-aware `clang` for ring |

Environment (PowerShell):

```powershell
$env:ZIG = "C:\path\to\zig-x86_64\zig.exe"
$env:LIBCLANG_PATH = "C:\path\to\llvm\bin"
$env:RUSTFLAGS = "-C target-feature=+crt-static"
$env:PATH = "C:\path\to\llvm\bin;$env:PATH"   # LLVM bin must be on PATH for ring
```

## Build

```powershell
git submodule update --init --recursive   # vendor/ghostty (fresh upstream)
cargo build --release --locked            # → target\release\herdr-app.exe
```

`vendor/ghostty` builds libghostty-vt through Zig
(`zig build -Demit-lib-vt -Doptimize=ReleaseFast -Dcpu=baseline
-Dtarget=aarch64-windows-msvc`).

## Install

```powershell
pwsh -NoProfile -ExecutionPolicy Bypass -File scripts\install-windows.ps1
# or: just --shell pwsh --shell-arg -Command install
```

Installs to `%LOCALAPPDATA%\Programs\herdr-app\` + Start Menu shortcut + user
PATH.

## Pairing with herdr-core

The app needs a herdr-core binary. Upstream only publishes
`windows-x86_64`; build herdr-core natively for ARM64 (see that repo's
`docs/windows-arm64.md`) and install it to
`%LOCALAPPDATA%\Programs\herdr-core\` with its `conpty\` bundle — hc loads
`conpty\conpty.dll` beside the exe because inbox ConPTY swallows OSC 7501
responses.

## Limitations

- No auto-update: the updater looks for `windows-aarch64` release manifests
  that upstream does not publish.
- Requires the x86_64 Zig under emulation for all Zig build steps.
