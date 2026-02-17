# Mojo LSP Setup for Neovim on NixOS

This document explains how the Mojo LSP is configured to work with Neovim through direnv.

## How It Works

### The Problem
- Mojo and its LSP server (`mojo-lsp-server`) are installed by pixi in `.pixi/envs/default/bin/`
- This directory is NOT in your PATH by default
- Neovim's lspconfig expects `mojo-lsp-server` to be in PATH
- Without it in PATH, the LSP won't start

### The Solution
We use **direnv** to automatically activate the pixi environment when you enter this directory:

1. **Nix provides**: CUDA toolkit, system libraries (including `libbsd`), and pixi
2. **direnv loads**: The Nix environment when you `cd` into the directory
3. **pixi shell-hook**: Adds `.pixi/envs/default/bin` to PATH
4. **Result**: `mojo-lsp-server` is in PATH, Neovim LSP works!

### Configuration

#### `.envrc` (direnv configuration)
```bash
# Load Nix FHS environment (provides CUDA, system libs, pixi)
use flake

# Activate pixi environment to expose mojo, mojo-lsp-server, etc. to PATH
watch_file pixi.toml
eval "$(pixi shell-hook)"
```

#### `flake.nix` updates
- Added `libbsd` to system libraries (fixes `mojo-lldb` debugger)
- FHS environment provides all necessary libraries for pixi packages

#### Neovim LSP config
Your existing `~/.config/nvim/lua/plugins/lsp.lua` already has:
```lua
mojo = {},
```

This tells nvim-lspconfig to:
- Look for `mojo-lsp-server` in PATH
- Start it automatically for `.mojo` files
- No additional configuration needed!

## Testing the Setup

Once direnv finishes loading the environment:

### 1. Check direnv is active
```bash
cd ~/code/gpu/mojo-gpu-puzzles
# You should see direnv loading message
```

### 2. Verify mojo-lsp-server is in PATH
```bash
which mojo-lsp-server
# Should output: /home/ericus/code/gpu/mojo-gpu-puzzles/.pixi/envs/default/bin/mojo-lsp-server

mojo-lsp-server --version
# Should output: LLVM version info (no errors)
```

### 3. Verify mojo-lldb works (debugger)
```bash
mojo-lldb --version
# Should output: lldb version info (no "libbsd.so.0 not found" error)
```

### 4. Test in Neovim
```bash
nvim problems/p01/p01.mojo
```

Expected behavior:
- LSP should start automatically
- You should see LSP features:
  - Hover (`<leader>k`) for documentation
  - Go to definition (`gd`)
  - Code actions (`<leader>ca`)
  - Diagnostics if there are errors
- Check LSP is running: `:LspInfo` should show `mojo` as attached

### 5. Test LSP functionality
In a `.mojo` file, try:
- Hover over a function or type
- Jump to definition
- See if syntax errors are highlighted
- Check autocomplete works

## Troubleshooting

### LSP not starting in Neovim?

**Check if mojo-lsp-server is in PATH:**
```bash
# In the project directory
which mojo-lsp-server
echo $PATH | tr ':' '\n' | grep pixi
```

**Check direnv is loaded:**
```bash
env | grep PIXI
# Should show several PIXI_* variables
```

**Reload direnv:**
```bash
direnv reload
```

**Check Neovim LSP status:**
```vim
:LspInfo
```

### "libbsd.so.0 not found" error?

This should be fixed now, but if you still see it:

**Check LD_LIBRARY_PATH includes libbsd:**
```bash
echo $LD_LIBRARY_PATH | tr ':' '\n' | grep libbsd
```

**Test mojo-lldb:**
```bash
ldd $(which mojo-lldb) | grep libbsd
# Should show: libbsd.so.0 => /nix/store/.../libbsd.so.0
```

### direnv takes too long to load?

First time builds all the Nix packages (CUDA toolkit, etc.). Subsequent loads are fast because everything is cached.

**Check build progress:**
```bash
# In another terminal
journalctl -f | grep nix
```

### Want to work without direnv?

You can always use pixi directly:
```bash
pixi run nvim problems/p01/p01.mojo
# OR
pixi shell
nvim problems/p01/p01.mojo
```

## Technical Details

### What `pixi shell-hook` does:
1. Adds `.pixi/envs/default/bin` to the front of PATH
2. Sets environment variables: `CONDA_PREFIX`, `PIXI_PROJECT_ROOT`, etc.
3. Sources conda activation scripts for packages
4. Makes all pixi-installed tools available

### Why direnv + pixi?
- **direnv**: Automatically loads environment when you enter the directory
- **pixi**: Manages Mojo, Python, and other project dependencies
- **Nix**: Provides system-level dependencies (CUDA, libraries)

This hybrid approach gives you:
- ✅ Automatic activation (direnv)
- ✅ Mojo ecosystem (pixi)
- ✅ NixOS compatibility (Nix flake)
- ✅ LSP works in Neovim
- ✅ All mojo tools in PATH

### Environment variables set:
- `PATH`: `.pixi/envs/default/bin` added first
- `PIXI_PROJECT_ROOT`: Project directory
- `CONDA_PREFIX`: Pixi environment location
- `CUDA_PATH`: CUDA toolkit from Nix
- `LD_LIBRARY_PATH`: System libraries from Nix + conda libraries

## Reusing in Other Projects

To use this setup in other Mojo projects:

1. **Copy the flake files:**
   ```bash
   cp flake.nix flake.lock other-mojo-project/
   ```

2. **Create `.envrc`:**
   ```bash
   echo 'use flake' > other-mojo-project/.envrc
   echo 'watch_file pixi.toml' >> other-mojo-project/.envrc
   echo 'eval "$(pixi shell-hook)"' >> other-mojo-project/.envrc
   ```

3. **Allow direnv:**
   ```bash
   cd other-mojo-project
   direnv allow
   ```

4. **Done!** LSP will work automatically.

Or, add to your shellenvs repo and reference it:
```bash
# In other project's .envrc
use flake path/to/shellenvs/mojo
eval "$(pixi shell-hook)"
```

## Next Steps

Once the LSP is working:
- Explore Mojo language features
- Use LSP for faster development
- Debug with `mojo-lldb` (now works with libbsd!)
- Try the interactive book: `pixi run book`

Happy coding! 🔥
