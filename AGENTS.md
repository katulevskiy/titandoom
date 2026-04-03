# AGENTS.md

## Cursor Cloud specific instructions

### Overview

This is **titandoom** — a C++23 3D multiplayer FPS using SDL3, EnTT (ECS), and GLM. It produces two executables from a single CMake build:

- `titandoom` — game client (requires GPU or software Vulkan at runtime)
- `titandoom-server` — headless dedicated server (no GPU needed)

Build/lint/run commands are in `README.md`. Key commands:

```
cmake --preset debug && cmake --build --preset debug   # configure + build
cmake --build --preset debug --target format-check     # lint (clang-format)
find src -name "*.cpp" | xargs clang-tidy -p build/debug  # static analysis
```

### Cloud VM caveats

- **No hardware GPU**: The cloud VM has no physical GPU. The game client requires `mesa-vulkan-drivers` (lavapipe) for software Vulkan rendering. Launch the client with:
  ```
  VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/lvp_icd.json DISPLAY=:1 \
    LSAN_OPTIONS=suppressions=sanitizers/lsan.supp ./build/debug/titandoom
  ```
- **Server runs headless**: The dedicated server has no GPU requirement:
  ```
  LSAN_OPTIONS=suppressions=sanitizers/lsan.supp ./build/debug/titandoom-server
  ```
- **ASan/UBSan leaks**: Debug builds enable sanitizers. SDL3's Linux backends trigger intentional one-time allocations that LeakSanitizer reports; use `LSAN_OPTIONS=suppressions=sanitizers/lsan.supp` to suppress them.
- **libclang-rt-18-dev** and **libstdc++-14-dev** are required for linking with ASan/UBSan on clang-18 in this environment (not covered by `scripts/setup-linux.sh`).
- **glslang-tools** provides the `glslangValidator` shader compiler needed by CMake; also not in the setup script.
- **Git hooks**: CMake configure auto-sets `core.hooksPath` to `.githooks/` (pre-commit auto-formats, pre-push blocks unformatted code).
