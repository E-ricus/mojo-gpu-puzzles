# NixOS Quick Start Guide

This is a **hybrid setup** for NixOS users: Nix provides system dependencies, pixi manages Mojo.

## TL;DR

```bash
# 1. Enter the Nix environment (FHS for compatibility - launches bash)
nix develop

# 2. Install Mojo and project dependencies with pixi
pixi install

# 3. Start solving puzzles!
pixi run p01
```

**Note:** The default environment uses FHS for maximum binary compatibility with pixi packages, which requires bash. If you prefer to stay in your shell (nushell, fish, etc.), see the "Light Shell" option below.

## What You Get

### From Nix (`flake.nix`)
- ✅ CUDA Toolkit 12.x
- ✅ GPU driver integration  
- ✅ System libraries (libGL, X11, etc.)
- ✅ FHS environment for binary compatibility
- ✅ Pixi package manager

### From Pixi (`pixi.toml`)
- ✅ Mojo compiler & MAX runtime
- ✅ Python 3.12 environment
- ✅ All project dependencies
- ✅ Development tools (mdbook, pre-commit, etc.)

## Setup Options

### Option 1: Manual (recommended first time)

```bash
nix develop
pixi install
```

### Option 2: Light shell (preserves your shell - nushell, fish, etc.)

If you prefer to stay in your current shell and don't need full FHS compatibility:

```bash
nix develop .#light
pixi install
```

**Note:** This may have binary compatibility issues with some pixi packages. If you encounter errors, use the default FHS shell instead.

### Option 3: direnv (with light shell)

For automatic environment loading that preserves your shell:

1. Edit `.envrc` to use the light shell:
   ```bash
   use flake .#light
   ```

2. Allow direnv:
   ```bash
   direnv allow
   pixi install
   ```

## Running Puzzles

Inside the Nix development environment:

```bash
# Individual puzzles
pixi run p01
pixi run p02
# ... up to p34

# Run all tests
pixi run tests

# Build and serve the interactive book
pixi run book

# Get your GPU specifications
pixi run gpu-specs
```

## Understanding the Setup

### Why the hybrid approach?

1. **Mojo isn't in nixpkgs**: Mojo is proprietary and only available through Modular's conda/pip channels
2. **GPU support on NixOS**: CUDA libraries need proper Nix packaging for NixOS
3. **Best of both worlds**: Nix for reproducible system deps, pixi for Mojo ecosystem

### What happens when you run `nix develop`?

1. Downloads/builds CUDA toolkit (~2GB, cached after first time)
2. Sets up FHS environment with proper library paths
3. Makes `pixi` available
4. Configures GPU access and CUDA environment variables
5. **Launches bash** (for FHS compatibility)

### What happens when you run `pixi install`?

1. Downloads Mojo compiler and MAX runtime from Modular
2. Installs Python packages and dependencies
3. Sets up project-specific tools
4. Creates `.pixi` environment directory (gitignored)

## Troubleshooting

### First build takes a long time?

Yes, downloading CUDA packages can take 5-15 minutes on first run. They're cached for future use.

### Can't find GPU?

```bash
# Check GPU is visible (should work outside Nix too)
nvidia-smi

# Inside Nix environment, verify CUDA
echo $CUDA_PATH
```

### Pixi installation fails?

Make sure you're inside the Nix environment first:
```bash
nix develop
# Now pixi has access to all system libraries
pixi install
```

### Library not found errors?

The default FHS shell should handle most library issues. If you're using the light shell (`nix develop .#light`) and encounter errors:

```bash
# Switch to the default FHS environment
nix develop
```

The FHS shell provides a complete `/lib`, `/lib64` structure that some binaries expect.

## GPU Support

### NVIDIA (your setup)
- Default environment, CUDA 12.x
- Includes nvidia-smi, compute-sanitizer, cuda-gdb

### AMD GPUs
```bash
pixi install -e amd
```

### Apple Silicon
Not applicable on NixOS, but the flake documents it for macOS users.

## Next Steps

1. Visit [puzzles.modular.com](https://puzzles.modular.com/) for the interactive tutorial
2. Read [NIX_SETUP.md](NIX_SETUP.md) for detailed information
3. Check [README.md](README.md) for the main project documentation

## File Overview

- `flake.nix` - Nix development environment (CUDA, system libs)
- `flake.lock` - Locked Nix dependencies
- `.envrc` - direnv configuration (optional)
- `pixi.toml` - Pixi project config (Mojo, Python, tasks)
- `.pixi/` - Pixi environment (gitignored, created by `pixi install`)

## Common Commands Reference

```bash
# Environment
nix develop              # Enter FHS shell (launches bash, full compatibility)
nix develop .#light      # Enter light shell (preserves your shell)
nix flake update         # Update Nix dependencies

# Pixi
pixi install             # Install all dependencies
pixi run tests           # Run test suite
pixi run format          # Format code
pixi run book            # Build and serve book
pixi run p01             # Run puzzle 01
pixi shell               # Enter pixi shell (nested in Nix)

# GPU Tools
nvidia-smi               # GPU info
pixi run gpu-specs       # Detailed GPU specs
pixi run memcheck p01    # Run memory checker on puzzle
```

Enjoy solving GPU puzzles! 🔥🧩
