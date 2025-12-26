# Neovim Plugin Documentation

Quick reference for tinkering with your config.

## bufferline.nvim

### Setup
```lua
require("bufferline").setup{
  options = {
    mode = "buffers",  -- or "tabs"
    numbers = "none",  -- or "ordinal", "buffer_id", "both"
    diagnostics = "nvim_lsp",  -- or "coc"

    -- Custom diagnostics indicator
    diagnostics_indicator = function(count, level)
      local icon = level:match('error') and ' ' or ' '
      return ' ' .. icon .. count
    end,

    -- Offset for file explorer
    offsets = {
      {
        filetype = 'neo-tree',
        text = 'File Explorer',
        text_align = 'center',
        separator = true,
      },
    },

    -- Sorting
    sort_by = 'extension',  -- or 'directory', custom function

    -- Styling
    separator_style = 'thin',  -- or 'slant', 'slope', 'thick', 'padded_slant'
  },
}
```

### Commands
**Navigation:**
- `:BufferLineCycleNext` - Next buffer
- `:BufferLineCyclePrev` - Previous buffer
- `:BufferLineGoToBuffer N` - Go to Nth buffer (negative = from end)
- `:BufferLinePick` - Interactive pick with chars

**Management:**
- `:BufferLineCloseLeft` - Close all left of current
- `:BufferLineCloseRight` - Close all right of current
- `:BufferLineCloseOthers` - Close all except current
- `:BufferLinePickClose` - Interactive close
- `:BufferLineMoveNext` - Move buffer right
- `:BufferLineMovePrev` - Move buffer left

**Sorting:**
- `:BufferLineSortByExtension`
- `:BufferLineSortByDirectory`
- `:BufferLineSortByTabs`

**Groups:**
- `:BufferLineGroupClose NAME` - Close group
- `:BufferLineGroupToggle NAME` - Toggle group visibility

**Tabs:**
- `:BufferLineTabRename [NAME]` - Rename tab

### Help
`:h bufferline-configuration`

---

## lazy.nvim Plugin Spec

### Complete Field Reference

```lua
{
  -- Source
  "owner/repo",  -- [1] Short URL (GitHub)
  dir = "/path/to/local/plugin",
  url = "https://git.example.com/repo.git",
  name = "custom-name",
  dev = false,

  -- Loading
  dependencies = { "dep1", "dep2" },
  enabled = true,  -- or function()
  cond = true,     -- or function() (disable without uninstall)
  priority = 50,   -- load order for startup plugins

  -- Configuration
  init = function() end,  -- runs at startup
  opts = {},              -- passed to setup() - PREFER THIS
  config = function() end,  -- or true to call setup()
  main = "module.name",

  -- Build
  build = "make",  -- or function() or table

  -- Lazy Loading
  lazy = true,
  event = "VimEnter",  -- or array or function
  cmd = "CommandName",  -- or array or function
  ft = "lua",          -- or array or function
  keys = "<leader>x",  -- or array or function

  -- Version Control
  branch = "main",
  tag = "v1.0.0",
  commit = "abc123",
  version = "^1.0.0",  -- semver
  pin = false,
  submodules = true,

  -- Advanced
  optional = false,
  specs = {},
  module = false,  -- prevent auto module loading
  import = "custom.plugins",
}
```

### keys Field Examples
```lua
-- Simple
keys = "<leader>ff"

-- With mode
keys = {
  { "<leader>ff", "<cmd>Telescope find_files<cr>", desc = "Find Files" },
  { "<leader>fg", "<cmd>Telescope live_grep<cr>", mode = "n" },
}

-- Function
keys = function()
  return { "<leader>x", custom_func }
end
```

### Best Practice
Use `opts` instead of `config` when possible:
```lua
-- Good
{ "plugin/name", opts = { option = true } }

-- Only if needed
{ "plugin/name", config = function()
  require('plugin').setup({ option = true })
end }
```

---

## telescope.nvim

### Builtin Pickers

**Files:**
- `find_files` - All files (respects .gitignore)
- `git_files` - Git-tracked files
- `live_grep` - String search (requires ripgrep)
- `grep_string` - Search word under cursor
- `oldfiles` - Recent files
- `buffers` - Open buffers

**Vim:**
- `help_tags`, `commands`, `keymaps`
- `quickfix`, `loclist`, `jumplist`
- `marks`, `registers`
- `colorscheme`, `vim_options`, `filetypes`

**Git:**
- `git_commits`, `git_bcommits` (buffer commits)
- `git_branches`, `git_status`, `git_stash`

**LSP:**
- `lsp_references`, `lsp_definitions`, `lsp_implementations`
- `lsp_document_symbols`, `lsp_workspace_symbols`
- `lsp_type_definitions`
- `diagnostics`

### Configuration
```lua
require('telescope').setup{
  defaults = {
    -- Global settings for all pickers
    mappings = {
      i = {  -- insert mode
        ["<C-j>"] = actions.move_selection_next,
        ["<C-k>"] = actions.move_selection_previous,
      },
      n = {  -- normal mode
        ["q"] = actions.close,
      },
    },
    layout_strategy = 'horizontal',  -- or 'vertical', 'center', 'cursor'
    layout_config = {
      width = 0.8,
      height = 0.8,
    },
  },
  pickers = {
    -- Per-picker overrides
    find_files = {
      theme = "dropdown",
      initial_mode = 'insert',
    },
    lsp_references = {
      initial_mode = 'normal',
    },
  },
  extensions = {
    -- Extension configs
  },
}
```

### Default Keymaps (in picker)
- `<C-n>/<C-p>` or `↓/↑` - Navigate
- `<C-x>/<C-v>/<C-t>` - Open in split/vsplit/tab
- `<Tab>/<S-Tab>` - Toggle selection
- `<C-q>/<M-q>` - Send to quickfix
- `<C-/>` or `?` - Show help (insert/normal)

### Themes
```lua
local builtin = require('telescope.builtin')
local themes = require('telescope.themes')

builtin.find_files(themes.get_dropdown())
builtin.find_files(themes.get_cursor())
builtin.find_files(themes.get_ivy())
```

### Custom Keymaps
```lua
vim.keymap.set('n', '<leader>ff', function()
  builtin.find_files({ cwd = vim.fn.getcwd() })
end, { desc = 'Find Files' })
```

### Extensions
```lua
-- Load extension
pcall(require('telescope').load_extension, 'fzf')

-- Use extension
:Telescope fzf
```

---

## Common Patterns in Your Config

### Get Project Root
```lua
local function get_project_root()
  local git_root = vim.fn.system('git rev-parse --show-toplevel 2>/dev/null')
  if vim.v.shell_error == 0 then
    return git_root:gsub('\n$', '')
  end
  return vim.fn.getcwd()
end
```

### Smart Buffer Close
```lua
local function smart_close_buffer()
  local bufs = vim.fn.getbufinfo({buflisted = 1})
  if #bufs <= 1 then
    vim.cmd('quit')
  else
    vim.cmd('bdelete')
  end
end
```

### Multi-mode Keymaps
```lua
local modes = { 'n', 'x', 'v' }
for i = 1, #modes do
  vim.keymap.set(modes[i], '<leader>x', action, { desc = 'description' })
end
```

### LSP Attach Keymaps
```lua
vim.api.nvim_create_autocmd('LspAttach', {
  callback = function(event)
    local map = function(keys, func, desc)
      vim.keymap.set('n', keys, func, { buffer = event.buf, desc = 'LSP: ' .. desc })
    end

    map('gd', require('telescope.builtin').lsp_definitions, 'Goto Definition')
  end,
})
```

---

## Helpful Vim Commands

**Buffers:**
- `:ls` - List buffers
- `:bn`, `:bp` - Next/prev buffer
- `:bd [N]` - Delete buffer N
- `:b [N]` - Switch to buffer N
- `:bufdo {cmd}` - Run cmd on all buffers

**Windows:**
- `<C-w>s` - Horizontal split
- `<C-w>v` - Vertical split
- `<C-w>hjkl` - Navigate windows
- `<C-w>=` - Equal window sizes

**Tabs (vim native):**
- `:tabnew` - New tab
- `:tabc` - Close tab
- `gt`, `gT` - Next/prev tab
- `:tabdo {cmd}` - Run cmd on all tabs

**Help:**
- `:h option-list` - All options
- `:h vim.keymap.set` - Keymap API
- `:h autocmd-events` - Autocommand events
- `:h lua-guide` - Lua guide
