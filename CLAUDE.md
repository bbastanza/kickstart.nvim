# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Neovim kickstart config - single-file config w/ lazy.nvim plugin manager. Personal fork w/ custom keymaps + settings.

## Architecture

### Core Structure
- `init.lua`: Single-file config containing all settings, keymaps, plugin specs
- `lua/kickstart/plugins/`: Optional kickstart plugins (debug, indent_line, lint) - commented out by default
- `lua/custom/plugins/`: User plugins (ng.nvim for Angular navigation)
- Plugin manager: lazy.nvim (auto-bootstraps on first run)

### Plugin Organization
All plugins defined in `require('lazy').setup({...})` call in init.lua:
- File explorer: neo-tree (toggle w/ `<leader>e`, focus w/ `<leader>o`)
- LSP: nvim-lspconfig + mason + mason-lspconfig + mason-tool-installer
- Completion: nvim-cmp + LuaSnip
- Fuzzy finding: telescope.nvim + telescope-fzf-native
- Treesitter: syntax highlighting/parsing + rainbow-delimiters (colored brackets) + nvim-ts-autotag (auto-close HTML/JSX tags)
- UI: which-key, mini.nvim (statusline, surround, ai textobjects), bufferline, gitsigns
- Formatting: conform.nvim (stylua for Lua)
- Theme: tokyonight + onedarkpro (onedark_vivid active w/ custom HTML/TS highlights)

### Custom Keymaps (leader = space)
Heavy customization in lines 157-187 applied to n/x/v modes:
- Window nav: `<C-hjkl>` instead of `<C-w>hjkl`
- Splits: `<leader>v` (vsplit), `<leader>b` (hsplit)
- Movement: `HJKL` = 10x/5x movement
- File ops: `<leader>fs` (save), `<leader>q` (quit), `<leader>x` (save+quit)
- Line manipulation: `<leader>jk` (insert empty lines)
- Custom: `<space>;` = `$`, `<leader>c` = toggle case, `<BS>` = X, `Y` = y$
- Tab nav: `<leader>t` (next), `<leader><BS>` (close)
- Angular: `<leader>at` (goto template), `<leader>ac` (goto component), `<leader>aT` (show TCB)

### LSP Config
- Auto-attach keymaps on LspAttach event (lines 452-523)
- Custom: `gk` for hover (not K), `gd/gr/gI/gD` for navigation
- Mason auto-installs LSP servers defined in `servers` table:
  - TypeScript/JavaScript: tsserver
  - Angular: angularls
  - Tailwind CSS: tailwindcss
  - Python: pyright
  - C#: omnisharp
  - Lua: lua_ls

### Settings
Notable custom opts (lines 58-72):
- `autochdir = false`: Keep cwd stable for project-wide searches
- Tab: 4 spaces
- `cmdheight = 1`, `laststatus = 1`: Minimal UI
- Font: Ubuntu Mono Nerd Font

## Common Tasks

### File Explorer
```
<leader>e  # Toggle neo-tree sidebar
<leader>o  # Focus neo-tree
```
Within neo-tree: use `?` to see all keymaps

### Code Navigation (LSP)
```
gd         # Go to definition
gr         # Go to references (Telescope)
gI         # Go to implementation
gD         # Go to declaration
gk         # Hover documentation
<leader>ca # Code action
<leader>rn # Rename symbol
<leader>ds # Document symbols
<leader>ws # Workspace symbols
[d / ]d    # Prev/next diagnostic
<leader>cd # Show diagnostic float
<leader>cl # Open diagnostic list
```

### Plugin Management
```bash
nvim  # Lazy auto-installs on startup
:Lazy  # View plugin status
:Lazy update  # Update all plugins
```

### LSP/Tools
```bash
:Mason  # Manage LSP servers/tools
:checkhealth  # Diagnose issues
```

### Testing Config Changes
- Edit init.lua
- Restart nvim (re-sourcing not supported w/ lazy.nvim)
- Check `:Lazy` and `:checkhealth` for issues

### Adding Custom Plugins
1. Add spec to init.lua's `require('lazy').setup({...})` OR
2. Create file in `lua/custom/plugins/` (must return plugin spec table)

## External Dependencies

Required:
- Neovim stable/nightly
- git, make, unzip, gcc/C compiler
- ripgrep (for telescope grep)

Recommended:
- Nerd Font (set `vim.g.have_nerd_font = true` in init.lua)

Language-specific:
- stylua (auto-installed by Mason for Lua formatting)
- Add LSP servers to `servers` table in init.lua (lines 541-569)

## Recent Enhancements

- **ng.nvim plugin**: Angular component ↔ template navigation (`<leader>at/ac/aT`)
- **Project-wide Telescope searches**: All search keymaps (`<leader>sf/sg/sw/s/`) now search from git root, not current file's dir. Fallback to cwd if not in git repo.
- Rainbow brackets (different colors per nesting level) for all languages
- Auto-closing HTML/JSX/TSX tags
- Enhanced HTML/TS syntax highlighting with custom colors
- Sidebar file tree (neo-tree)
- LSP for TypeScript/JavaScript, Angular, Tailwind CSS, Python, C#
- Telescope opens LSP pickers in normal mode (no stray input)

## Notes

- Kickstart philosophy: single-file, fully documented starting point (not a distro)
- For modular structure: see kickstart-modular.nvim fork
- Clipboard NOT synced to OS by default (line 88 commented)
- Format-on-save enabled via conform.nvim (disabled for C/C++)
