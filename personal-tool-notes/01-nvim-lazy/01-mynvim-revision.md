# Neovim config revise

Personal cheatsheet for this setup. Read this after time away.

- **Machine path:** `~/.config/nvim`
- **GitHub:** [github.com/kitparl/neovim](https://github.com/kitparl/neovim)
- **Stack:** Neovim + LazyVim + lazy.nvim
- **Last revised:** 23 Aug 2026
- **Neovim:** v0.12.4
- **LazyVim:** 16.0.0
- **Leader key:** `Space`

If an AI (or you) changes plugins, keymaps, LSP, extras, or layout, **update this file in the same change**.

---

## 1. What this setup is for

You use Neovim as a daily IDE, mainly for:

- **Java / Maven** (nvim-java, maven.nvim, custom Maven terminal)
- **Go** (vim-go)
- **Ruby** (ruby-lsp via asdf, Mason disabled for that server)
- **Web** (live-server for HTML/CSS/JS)
- **Markdown** (render-markdown, markdown extra)
- **DSA / LeetCode** (leetcode.nvim)
- **AI complete** (Codeium.vim, not codeium.nvim)
- **Claude Code** (`coder/claudecode.nvim` via LazyVim extra)
- **Cursor agent** (`avante.nvim` + Cursor CLI ACP, billed to your Cursor subscription)

Format-on-save is **off** (`vim.g.autoformat = false` in `init.lua`).

Theme is **rose-pine dawn** (light), with Tokyo Night disabled.

---

## 2. File structure

```
~/.config/nvim/
├── init.lua                 # entry: lazy, title, autocommands, autoformat off
├── lazy-lock.json           # pinned plugin commits (do not edit by hand)
├── lazyvim.json             # LazyVim extras + version
├── stylua.toml              # Lua formatter: 2 spaces, 120 cols
├── .neoconf.json            # lua_ls / neodev for this config
├── .gitignore
├── revise.md                # this file
├── ai-connect.md            # Claude Code + Cursor CLI login / tokens
├── readme.md                # leftover template; ignore for daily use
├── .cursor/rules/           # Cursor rules so AI updates revise.md
│   └── update-revise.mdc
└── lua/
    ├── autocommands.lua     # extra autocmds (Maven auto-run is commented)
    ├── config/
    │   ├── lazy.lua         # bootstraps lazy.nvim + LazyVim
    │   ├── options.lua      # extra vim options; prepends ~/.local/bin to PATH
    │   ├── keymaps.lua      # YOUR keymaps (plus LazyVim defaults)
    │   ├── autocmds.lua     # extra LazyVim autocmds (empty)
    │   └── vim-css-color.lua
    ├── plugins/             # every .lua file here is a lazy.nvim spec
    │   ├── avante.lua       # Cursor subscription via ACP (keys: <leader>A)
    │   ├── claudecode.lua   # Claude CLI path + missing-binary notify
    │   ├── theme.lua
    │   ├── codium.lua
    │   ├── nvim_devdocs.lua
    │   ├── nvim_tree.lua
    │   ├── lazy_git.lua
    │   ├── git_blame.lua
    │   ├── live_server.lua
    │   ├── leetcode.lua
    │   ├── lsp.lua          # `gl` diagnostic float; must return {}
    │   ├── nvim_cmp.lua     # disabled (returns {}); blink.cmp is default
    │   ├── ruby_lsp.lua
    │   ├── intend.lua       # indent guides + rainbow delimiters
    │   ├── starter.lua      # mini.starter disabled
    │   ├── multiple_cursors.lua
    │   ├── nvim_ufo.lua     # folds
    │   ├── resolve_git_conflicts.lua
    │   ├── vim_go.lua
    │   ├── maven.lua
    │   ├── render_markdown.lua
    │   └── java/
    │       ├── init.lua     # nvim-java
    │       └── maven.lua    # old comments only
    └── run_program/
        ├── run_program.lua  # <leader>z run current file
        └── maven_run.lua    # Maven bottom terminal
```

**Rule:** every file under `lua/plugins/` must `return { ... }` or `return {}`. A `nil` return crashes startup (`Invalid spec module`).

Plugins themselves live in `~/.local/share/nvim/lazy/`, not in this repo.

---

## 3. How startup works

1. `init.lua` loads `config.lazy`.
2. `lazy.lua` clones lazy.nvim if missing, then:

```lua
require("lazy").setup({
  spec = {
    { "LazyVim/LazyVim", import = "lazyvim.plugins" },
    { import = "plugins" },
  },
  defaults = { lazy = false, version = false },
})
```

3. LazyVim extras come from `lazyvim.json`.
4. Your specs in `lua/plugins/` override / add plugins.
5. `init.lua` then sets window title, loads `autocommands`, and disables autoformat.

---

## 4. LazyVim settings

From `lazyvim.json`:

| Extra | What it adds |
|---|---|
| `extras.ai.claudecode` | `coder/claudecode.nvim` — real Claude Code CLI inside Neovim |
| `extras.ai.codeium` | Codeium as LazyVim extra (you also override with `codeium.vim`) |
| `extras.coding.luasnip` | Snippet engine |
| `extras.coding.mini-surround` | Surround text objects (`sa`, `sd`, `sr`) |
| `extras.coding.yanky` | Yank history |
| `extras.dap.core` | Debug Adapter Protocol (nvim-dap) |
| `extras.editor.inc-rename` | Incremental LSP rename |
| `extras.editor.refactoring` | refactoring.nvim |
| `extras.lang.markdown` | Markdown LSP / preview extras |
| `extras.util.mini-hipatterns` | Hex color / TODO highlights |

Other important settings:

| Setting | Where | Value |
|---|---|---|
| Colorscheme | `lua/plugins/theme.lua` | `rose-pine` (dawn) |
| Autoformat | `init.lua` | `false` |
| Plugin versions | `lua/config/lazy.lua` | latest git (`version = false`) |
| Update checker | `lua/config/lazy.lua` | on, no notify |
| Custom plugins lazy? | `lua/config/lazy.lua` | no (`lazy = false`) |
| Completion | LazyVim 16 default | **blink.cmp** (nvim-cmp spec is empty) |
| Picker | LazyVim 16 default | **fzf-lua / snacks** (telescope still used by some plugins) |
| Treesitter | LazyVim 16 | `nvim-treesitter` **main** branch (needs `tree-sitter` CLI) |

Add / remove extras with `:LazyExtras` or by editing `lazyvim.json`.

---

## 5. Plugins we use (and why)

### Distro / UI

| Plugin | File | Why |
|---|---|---|
| LazyVim | distro | Defaults, extras, LSP, completion, which-key |
| lazy.nvim | `config/lazy.lua` | Plugin manager |
| rose-pine | `theme.lua` | Light dawn theme |
| indent-blankline | `intend.lua` | Indent guides |
| rainbow-delimiters | `intend.lua` | Rainbow brackets (pcall-wrapped; skipped on AI/terminal buffers) |
| mini.indentscope | `intend.lua` | Animated current-scope line |
| nvim-tree.lua | `nvim_tree.lua` | File tree (`:NvimTreeToggle`) |
| nvim-ufo | `nvim_ufo.lua` | Better folds |
| render-markdown | `render_markdown.lua` | Pretty markdown in-buffer |
| vim-css-color | `config/vim-css-color.lua` | Color previews in CSS/HTML/JS |
| mini.starter | `starter.lua` | **Disabled** |

### Coding / AI / tracking

| Plugin | File | Why |
|---|---|---|
| claudecode.nvim | LazyVim extra `ai.claudecode` + `claudecode.lua` | Claude Code CLI in a terminal; send buffers/selections |
| avante.nvim | `avante.lua` | Cursor subscription in Neovim (ACP). Keys: `<leader>A…` |
| codeium.vim | `codium.lua` | Inline AI; `codeium.nvim` is disabled |
| vim-visual-multi | `multiple_cursors.lua` | Multi-cursor (`Ctrl-n`) |
| leetcode.nvim | `leetcode.lua` | LeetCode in Neovim |

### Git

| Plugin | File | Why |
|---|---|---|
| lazygit.nvim | `lazy_git.lua` | TUI git (`<leader>lg`) |
| git-blame.nvim | `git_blame.lua` | Blame virtual text (off by default) |
| git-conflict.nvim | `resolve_git_conflicts.lua` | Conflict markers |

### Languages / tools

| Plugin | File | Why |
|---|---|---|
| nvim-java | `java/init.lua` | Java LSP / DAP / tests |
| maven.nvim | `maven.lua` | `:Maven` / `:MavenExec` (uses `./mvnw`) |
| vim-go | `vim_go.lua` | Go fmt via goimports, Go tools |
| ruby_lsp | `ruby_lsp.lua` | `~/.asdf/shims/ruby-lsp`, Mason off |
| live-server-nvim | `live_server.lua` | Serve HTML on port **8080** |
| nvim-devdocs | `nvim_devdocs.lua` | Offline docs; Java docs auto-installed |
| nvim-lspconfig | `ruby_lsp.lua` + LazyVim | Language servers |

### Custom Lua (not plugins)

| Module | Why |
|---|---|
| `run_program.run_program` | Run current Java/JS/Dart/Python/Go file in a float |
| `run_program.maven_run` | Bottom zsh + `mvn …`; auto `compile exec:java` on Java save if terminal is open |

---

## 6. How to add a new language LSP

LazyVim 16 uses **Mason** + **nvim-lspconfig** + **vim.lsp**.

### Fastest: a LazyVim language extra

```
:LazyExtras
```

Enable e.g. `lang.go`, `lang.python`, `lang.typescript`, `lang.java`, `lang.ruby`. Restart Neovim. Mason installs the servers.

Or edit `lazyvim.json` and add:

```json
"lazyvim.plugins.extras.lang.python"
```

Then `:Lazy sync`.

### Manual: Mason only

```
:Mason
```

- `i` install under cursor
- `/` search (`lua_ls`, `gopls`, `jdtls`, `pyright`, `tsserver` / `vtsls`, `rust_analyzer`, …)
- `u` update, `X` uninstall

Mason bin path is already on Neovim’s PATH.

### Manual: lspconfig spec (when Mason should not install)

Same pattern as Ruby. Create `lua/plugins/<lang>_lsp.lua`:

```lua
return {
  {
    "neovim/nvim-lspconfig",
    opts = {
      servers = {
        gopls = {},          -- Mason will install gopls
        pyright = {},
        ruby_lsp = {
          mason = false,     -- you install it yourself
          cmd = { vim.fn.expand("~/.asdf/shims/ruby-lsp") },
        },
      },
    },
  },
}
```

### Treesitter parser (syntax / folds)

LazyVim 16 uses treesitter **main**. Needs Homebrew `tree-sitter-cli`.

```
:TSInstall python
:TSUpdate
```

Or it auto-installs when you open a file of that type.

### Formatter / linter

- Format: conform.nvim via Mason (`:Mason`, install `stylua`, `prettier`, `gofumpt`, …)
- Lint: nvim-lint
- Your **global autoformat is off**. Format one buffer with LazyVim’s `<leader>cf`.

### Debug (DAP)

You already have `extras.dap.core`. For a language, install the adapter in `:Mason` (e.g. `js-debug-adapter`, `codelldb`) or enable the lang extra that wires DAP.

---

## 7. Keymaps

Leader = `Space`. Press `Space` and wait for **which-key**.

### Yours (custom)

| Keys | Mode | What |
|---|---|---|
| `<leader>z` | n | Run current file (java/js/dart/py/go) in a float |
| `<leader>mc` | n | `mvn clean install` in bottom terminal |
| `<leader>mt` | n | `mvn test` |
| `<leader>mr` | n | `mvn compile exec:java` |
| `<leader>md` | n | `mvn dependency:resolve` |
| `<leader>tt` | n | Toggle / shrink Maven terminal |
| `<leader>lt` | n | Toggle live-server (port 8080) |
| `<leader>rn` | n | IncRename (type new name after) |
| `<leader>dd` / `<leader>dc` | n | Delete inner word and insert |
| `<leader>cpf` | n | Copy full file path to clipboard |
| `<leader>cpp` | n | Copy directory path to clipboard |
| `<leader>sh` | n | Split horizontal |
| `<leader>sv` | n | Split vertical |
| `<leader>sx` | n | Close split |
| `<leader>so` | n | Only this split |
| `<leader>td` | n | Toggle diagnostics |
| `gl` | n | Diagnostic float (focusable) |
| `<leader>lg` | n | LazyGit |
| `<leader>gbt` | n | Toggle git blame |
| `<leader>gbo` | n | Open blame commit URL |
| `<leader>gbc` | n | Copy blame commit URL |
| `<leader>do` | n | Open DevDocs |
| `<leader>di` | n | Install DevDocs set |
| `<leader>ac` | n | Toggle Claude Code |
| `<leader>af` | n | Focus Claude Code |
| `<leader>ar` | n | Resume Claude Code |
| `<leader>aC` | n | Continue last Claude Code session |
| `<leader>ab` | n | Add current buffer to Claude |
| `<leader>as` | v | Send selection to Claude |
| `<leader>aa` | n | Accept Claude diff |
| `<leader>ad` | n | Deny Claude diff |
| `<leader>At` | n | Cursor (Avante) toggle |
| `<leader>Aa` | n/v | Cursor ask |
| `<leader>Ac` | n | Cursor chat |
| `<leader>An` | n | Cursor new chat |
| `<leader>Ae` | n/v | Cursor edit selection |
| `<leader>Af` | n | Cursor focus |
| `<leader>Ah` | n | Cursor history |
| `<leader>Am` | n | Cursor models |
| `<leader>Ap` | n | Cursor switch provider |
| `<leader>Ar` | n | Cursor refresh |
| `<leader>As` | n | Cursor stop |
| `<leader>ct` | n | Toggle Codeium |
| `<leader>cc` | n | Codeium Chat |
| `<leader>cs` | n | Codeium Status |
| `<Tab>` | i | Accept Codeium, else real Tab |
| `<C-g>` | i | Accept Codeium |
| `<C-;>` / `<C-,>` | i | Next / prev Codeium |
| `<A-]>` / `<A-[>` | i | Next / prev Codeium |
| `<C-x>` | i | Clear Codeium |
| `<C-f>` | i | Accept next Codeium word |
| `<C-l>` | i | Accept next Codeium line |
| `<C-\>` | i | Manually trigger Codeium |
| `zR` / `zM` | n | UFO open / close all folds |

Commands: `:OpenConfig`, `:OpenPlugin`, `:NvimTreeToggle`, `:Leet`, `:Maven`, `:LiveServerInstall`, `:ClaudeCode`, `:AvanteToggle`, `:AvanteAsk`.

### LazyVim defaults you will actually use

| Keys | What |
|---|---|
| `<leader><space>` | Find files |
| `<leader>/` | Grep (live) |
| `<leader>ff` | Files |
| `<leader>fr` | Recent files |
| `<leader>sg` | Grep |
| `<leader>ss` | LSP document symbols |
| `<leader>e` | Explorer (neo-tree / snacks) |
| `<leader>l` | Lazy plugin UI |
| `<leader>L` | LazyVim changelog |
| `<leader>cm` | Mason |
| `<leader>cf` | Format buffer |
| `<leader>cd` | Line diagnostics |
| `gd` `gr` `gI` `gy` `K` | Definition / refs / impl / type / hover |
| `<leader>ca` | Code action |
| `<leader>cr` | Rename |
| `]d` `[d` | Next / prev diagnostic |
| `]e` `[e` | Next / prev error |
| `<S-h>` / `<S-l>` | Prev / next buffer |
| `<leader>bd` | Delete buffer |
| `<C-h/j/k/l>` | Move between windows |
| `<leader>-` / `<leader>\|` | Split below / right |
| `<leader>wd` | Close window |
| `<C-s>` | Save |
| `<leader>qq` | Quit all |
| `<leader>ft` / `<C-/>` | Terminal |
| `<leader>gg` | Lazygit (root) |
| `<leader>uf` | Toggle format |
| `<leader>ud` | Toggle diagnostics (LazyVim) |
| `<leader>uw` | Toggle wrap |
| `<A-j>` / `<A-k>` | Move line |

### Git conflicts (in a conflicted buffer)

| Keys | What |
|---|---|
| `co` | Take ours |
| `ct` | Take theirs |
| `cb` | Take both |
| `c0` | Take none |
| `]x` `[x` | Next / prev conflict |

### Multi-cursor (vim-visual-multi)

| Keys | What |
|---|---|
| `Ctrl-n` | Add cursor on next match |
| `Ctrl-Up` / `Ctrl-Down` | Add cursor vertically |
| `q` / `Q` | Skip / back |
| `Esc` | Exit |

### Surround (mini.surround)

| Keys | What |
|---|---|
| `sa` | Add surround |
| `sd` | Delete surround |
| `sr` | Replace surround |

---

## 8. Daily workflows

### Run a single file

Open `Foo.java` / `app.py` / `main.go` / `index.js` / `main.dart` → `<leader>z`.

### Maven project

- `<leader>mc` compile/package
- `<leader>mt` tests
- `<leader>mr` run `exec:java`
- `<leader>tt` show/hide the bottom terminal  
  Saving a `.java` file re-runs `compile exec:java` **if that terminal is already open**.

### HTML live reload

Open the project → `<leader>lt` → browser on `http://0.0.0.0:8080`.

### Claude Code

Needs the **Claude Code CLI** on PATH (`claude`), then login once. Full steps: **`ai-connect.md`**.

- `<leader>ac` open/toggle Claude
- Visual-select code → `<leader>as` send it
- `<leader>ab` add the current file
- `<leader>aa` / `<leader>ad` accept or reject a diff

Keep Codeium for inline Tab-complete. Claude Code is for edits and questions.

If `claude` is missing:

```sh
# Install Claude Code CLI, then:
claude
```

### Cursor (your Cursor subscription)

Uses **Cursor CLI** (`agent`) over ACP, not the Cursor desktop app. Keys are **`<leader>A`** (capital A) so they do not clash with Claude Code (`<leader>a`). Full steps: **`ai-connect.md`**.

**One-time setup**

```sh
# 1. Install Cursor CLI (puts `agent` in ~/.local/bin)
curl https://cursor.com/install -fsSL | bash

# 2. Reload PATH, then log in with the same Cursor account as the desktop app
source ~/.zshrc
agent login
```

**In Neovim**

- `<leader>At` toggle Cursor sidebar
- `<leader>Aa` ask (visual select first for code context)
- `<leader>Ac` chat
- `<leader>Ae` edit selection
- `<leader>As` stop

If Avante says it cannot find `agent`, install the CLI and restart Neovim. This config prepends `~/.local/bin` and `~/.claude/local` to PATH in `lua/config/options.lua` so GUI Neovim still finds them.

Do **not** enable LazyVim extra `ai.avante` — it defaults to Copilot and steals Claude Code keymaps.

### Java docs inside Neovim

`<leader>di` if needed, then `<leader>do`. Java docs are in `ensure_installed`.

### First-time Codeium

```
:Codeium Auth     " if using the nvim extra; vim plugin uses its own login
```

---

## 9. Clone on a new machine

Needs: Neovim **>= 0.11.2** (this machine is 0.12.4), Git, a Nerd Font, ripgrep, fd, fzf, lazygit (optional), **tree-sitter-cli**, Claude Code CLI (`claude`) for `<leader>ac`, Cursor CLI (`agent`) for `<leader>A`.

```sh
brew install neovim ripgrep fd fzf lazygit tree-sitter-cli
git clone git@github.com-kitparl:kitparl/neovim.git ~/.config/nvim
# SSH host github.com-kitparl uses ~/.ssh/id_ed25519_kitparl
nvim
```

First launch runs `:Lazy sync`. Then `:Mason` if language servers are missing.

**GitHub SSH:** this Mac uses host `github.com-kitparl` (account **kitparl**). Default `github.com` SSH is **pranshu-bisht**. Do not mix them.

Push config changes:

```sh
cd ~/.config/nvim
git add -A
git commit -m "Describe what changed"
git push
```

---

## 10. Update Neovim / LazyVim / plugins

```sh
brew upgrade neovim
cd ~/.config/nvim
nvim --headless "+Lazy! sync" +qa
```

That rewrites `lazy-lock.json`. Commit it if you want GitHub to match.

Treesitter CLI (required since LazyVim 16):

```sh
brew install tree-sitter-cli
```

---

## 11. Things that bite you (from past breakage)

1. **`lua/plugins/*.lua` must return a table.** Commenting out the whole spec without `return {}` shows `Invalid spec module`.
2. **Do not `require("plugin").setup()` in `init.lua`.** Put setup in a lazy spec (`opts` / `config`).
3. Mini plugins are **`nvim-mini/...`**, not `echasnovski/...`.
4. Neovim 0.12: use `vim.diagnostic.is_enabled()` / `vim.diagnostic.enable(false)`, not `is_disabled()` / `disable()`.
5. Theme is **rose-pine** in `theme.lua`. Do not set LazyVim `colorscheme` to a tokyonight function — tokyonight is disabled.
6. `:OpenConfig` opens `~/.config/nvim/init.lua`.
7. nvim-devdocs `ensure_installed` can block startup while Java docs download. It is lazy-loaded via `cmd` / keys now.
8. Completion is **blink.cmp**. The old nvim-cmp + Codeium source file is a stub (`return {}`).
9. Claude Code is `<leader>a…`. Cursor/Avante is `<leader>A…`. Do not enable LazyVim extra `ai.avante`.

---

## 12. Useful commands

| Command | What |
|---|---|
| `:Lazy` | Plugins |
| `:Lazy sync` | Install / update / clean |
| `:LazyExtras` | Toggle extras |
| `:Mason` | LSP / DAP / linters / formatters |
| `:LspInfo` | Active language servers |
| `:checkhealth` | Diagnose Neovim |
| `:checkhealth nvim-treesitter` | Treesitter CLI / parsers |
| `:WhichKey` | Keymap helper |
| `:Inspect` | Highlight / TS under cursor |
| `:messages` | Recent errors |

---

## 13. AI / agent rule (Cursor, Claude Code, and others)

Any coding agent that edits this Neovim config **must** update `revise.md` in the same change when it:

- adds, removes, or replaces a plugin
- changes a keymap, command, or extra
- changes LSP / Mason / formatter / DAP setup
- changes file layout, theme, or global options (`autoformat`, leader, etc.)

**Source of truth:** `AGENTS.md`

| Agent | Where it reads the rule |
|---|---|
| Most agents (Codex, Cursor, Amp, …) | `AGENTS.md` |
| Claude Code | `CLAUDE.md` (imports AGENTS.md), `.claude/rules/` |
| Gemini CLI | `GEMINI.md` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Windsurf | `.windsurf/rules/update-revise.md` |
| Cline / Roo | `.clinerules` |
| Cursor | `.cursor/rules/update-revise.mdc` and `.cursorrules` |

User-wide copies (still apply when the agent is opened in another folder):

- `~/.cursor/rules/nvim-revise.mdc`
- `~/.claude/CLAUDE.md`
- `~/.codex/AGENTS.md`
