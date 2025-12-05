# Task: Create Cross-Platform Go Installer Build System

**ID**: 012
**Status**: new

## Context

Create a complete Go application build system for the smaqit installer. The installer CLI (`installer/main.go`) needs proper build scripts to compile into executables for multiple target platforms (Windows, macOS, Linux) with different architectures.

## Acceptance Criteria

- [ ] Create `Makefile` with targets for building all platforms
- [ ] Create `build.sh` (bash script) for Unix-like systems
- [ ] Create `build.bat` (batch script) for Windows systems
- [ ] Support target platforms:
  - Linux (amd64, arm64)
  - macOS (amd64, arm64)
  - Windows (amd64)
- [ ] Output binaries to `dist/` folder with platform-specific naming
- [ ] Include `clean` target to remove build artifacts
- [ ] Include `install` target for local installation
- [ ] Document build instructions in README or separate BUILD.md

## Notes

- Go supports cross-compilation via `GOOS` and `GOARCH` environment variables
- Consider using `go build -ldflags` to embed version info
- Installer version should sync with SMAQIT.md version (per copilot-instructions)
- Binary naming convention: `smaqit-{os}-{arch}[.exe]`
