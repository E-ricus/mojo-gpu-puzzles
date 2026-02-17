# NixOS Setup Guide

This repository includes a `flake.nix` for NixOS users to provide system-level dependencies while using `pixi` for Mojo/MAX installation.

## Quick Start

### 1. Enter the Development Environment

```bash
nix develop
```

This will:
- Provide CUDA toolkit and GPU drivers
- Install pixi and other system dependencies
- Set up FHS environment with proper library paths for NixOS
- **Launch bash** (required for FHS compatibility)

**Note:** The default shell is an FHS environment for maximum binary compatibility. If you prefer to stay in your shell (nushell, fish, etc.), there's a light shell available: `nix develop .#light` (may have compatibility issues with some pixi packages).

### 2. Install Mojo and Dependencies with Pixi

Once inside the Nix shell:

```bash
pixi install
```

This will install:
- Mojo compiler and MAX runtime
- Python packages
- Development tools (mdbook, pre-commit, etc.)

### 3. Start Working on Puzzles

```bash
# Run individual puzzles
pixi run p01
pixi run p02

# Run all tests
pixi run tests

# Build and serve the interactive book
pixi run book

# Get GPU information
pixi run gpu-specs
```

## Architecture

This setup uses a **hybrid approach**:

- **Nix/flake.nix** provides:
  - CUDA toolkit (12.x)
  - System libraries (libGL, X11, etc.)
  - GPU driver integration
  - FHS environment for binary compatibility
  - Development tools (git, bash, etc.)

- **Pixi/pixi.toml** provides:
  - Mojo compiler and MAX runtime (not available in nixpkgs)
  - Python environment and packages
  - Project-specific dependencies
  - Development task runners

## Why This Approach?

1. **Mojo Availability**: Mojo is proprietary and only distributed through Modular's conda/pip channels
2. **Binary Compatibility**: FHS environment ensures pixi-installed binaries work on NixOS
3. **GPU Support**: Proper CUDA toolkit and driver integration from Nix
4. **Official Support**: The repository is designed for pixi; this setup respects that

## Alternative Shells

### Light Shell (preserves your shell)

If you prefer to stay in your current shell (nushell, fish, zsh, etc.):

```bash
nix develop .#light
```

This is a standard mkShell that **preserves your shell** but may have binary compatibility issues with some pixi packages.

Use the default FHS shell if you see errors like "cannot find shared library" or "no such file or directory" for system libraries.

## Troubleshooting

### CUDA/GPU Issues

If you encounter GPU-related errors:

```bash
# Check GPU is visible
nvidia-smi

# Verify CUDA environment variables
echo $CUDA_PATH
echo $CUDA_HOME

# Check library paths
echo $LD_LIBRARY_PATH
```

### Pixi Installation Issues

If pixi has trouble installing packages:

1. Make sure you're inside the Nix development shell
2. Try clearing pixi cache:
   ```bash
   rm -rf .pixi
   pixi install
   ```

### Binary Compatibility Issues

If you're using the light shell and see errors about missing libraries:

1. Use the default FHS shell: `nix develop`
2. The FHS environment provides `/lib`, `/lib64`, etc., which many binaries expect
3. Note: The FHS shell launches bash (required for FHS compatibility)

## GPU Environments

This project supports multiple GPU types. Pixi will automatically select the right environment:

- **NVIDIA GPUs**: Uses `nvidia` environment (CUDA 12)
- **AMD GPUs**: Uses `amd` environment (ROCm 6.3)
- **Apple Silicon**: Uses `apple` environment (macOS 15+)

The default is NVIDIA. To explicitly select:

```bash
pixi install -e amd    # For AMD GPUs
pixi install -e nvidia # For NVIDIA GPUs (default)
```

## Next Steps

1. Visit the interactive tutorial at [puzzles.modular.com](https://puzzles.modular.com/)
2. Read the main [README.md](README.md) for project details
3. Start with `pixi run p01` and work through the puzzles!

## Notes

- The `.pixi` directory is gitignored and managed by pixi
- CUDA toolkit version is managed by the flake (currently 12.x)
- System libraries are provided by Nix to avoid NixOS path issues
