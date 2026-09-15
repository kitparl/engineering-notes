# My Revision Notes — Claude Code in Neovim + LazyVim Keymaps

Personal study sheet from today's session. Read this before you start coding.
This is a **personal notes file** (not the official cheatsheet — that's `revise.md`).

---

## PART 1 — Claude Code Integration (Q&A recap)

### What is it
This config has **real Claude Code CLI** wired into Neovim via `coder/claudecode.nvim`
(`lua/plugins/claudecode.lua`). It runs the actual `claude` binary in a terminal split,
connected to Neovim over a local WebSocket (same protocol as the VS Code extension).
This is **separate** from Cursor/Avante (`<leader>A…`), which uses the Cursor CLI (`agent`).

Login: Claude Pro/Max subscription (usual) or Anthropic API key (pay-per-token).
Full steps: `ai-connect.md`.

### Opening / switching sessions

| Key | What it does |
|---|---|
| `<leader>ac` | **Toggle** Claude — opens if closed, hides if open. Does NOT start a new session if one's already running. |
| `<leader>af` | **Focus** — jump cursor into the already-open Claude window |
| `<leader>ar` | **Resume** — opens a picker of ALL past sessions for this project, pick one to reopen |
| `<leader>aC` | **Continue** — reopens the single most recent session directly, no picker |

To start a **brand new session** while already inside Claude: type `/clear` (or `/new`) inside
the Claude prompt itself. `/clear` does NOT delete the old chat — it's still in `~/.claude/projects/`
and reachable later via `<leader>ar` or `/resume`.

### Adding files/code to context

| Key | Mode | What |
|---|---|---|
| `<leader>ab` | n | Add **entire current buffer/file** to Claude |
| `<leader>as` | v | Send the **visual selection** only |

To select the whole file first (if you want to visually select instead of `<leader>ab`): `ggVG`.

There is **no "unselect a file"** command — `<leader>ab` is a one-shot `@mention` for the next
prompt, not a persistent toggle. To remove its influence: `/compact` (summarize away) or `/clear`
(wipe chat).

### Models

Inside the Claude prompt (not a Neovim keymap):

```
/model            " opens interactive picker
/model sonnet     " daily coding (Pro default)
/model opus       " harder reasoning
/model haiku      " fast/cheap
/model opusplan   " Opus while planning, Sonnet while executing
```

`Enter` in the picker = save as default. `s` = this session only.

### Permission modes (cycle with `Shift+Tab` — must be in **terminal/insert mode**, not Neovim normal mode over the terminal buffer)

| Mode | Badge | Behavior |
|---|---|---|
| **Manual** (default) | `⏸ manual mode on` | Asks before every **first** use of a tool on a given file/command. Diff split opens in Neovim for file edits. |
| **Accept edits** | `⏵⏵ accept edits on` | File edits + common fs commands (`mkdir`, `mv`, etc.) auto-apply, no diff, no prompt. Shell commands still ask. |
| **Plan** | `⏸ plan mode on` | Read-only. Claude researches and writes a plan; no edits until you approve and it flips back to manual/accept-edits. |
| **Auto** (if available on your plan) | `⏵⏵ auto mode on` | Nearly everything auto-runs; only stops for genuinely risky stuff. |

Important nuance discovered today: **manual mode only asks on the FIRST use of a tool per file/path
per session.** Once you say "Yes" to editing `test.py`, further edits to `test.py` in the *same
session* won't re-prompt — even though the mode still says "manual". This is not a bug.

There's no Cursor-style Ask/Plan/Agent/Debug mode split — Claude Code only has the permission-mode
cycle above. Closest mapping:
- Cursor Ask → stay in Plan mode and just ask questions
- Cursor Plan → Plan mode
- Cursor Agent → Manual / Accept-edits
- Cursor Debug → **no equivalent**, just describe the bug in normal mode

### Reviewing / accepting Claude's edits

For **existing file edits** (Edit tool): Neovim auto-opens a real diff split (old vs new, editable)
AND the terminal shows a text menu:
```
1. Yes
2. Yes, and switch to accept edits (auto-approve) for this session
3. No
```
Approve with `1` in the terminal, or use the Neovim-side keys on the diff:

| Key | What |
|---|---|
| `<leader>aa` | Accept diff |
| `<leader>ad` | Deny/reject diff |

You can also **edit the proposed change directly** in the diff buffer before accepting — it's a
real editable buffer, not read-only.

For **brand-new files** (Write tool): there's only ONE step — "Do you want to create this file?"
with the full content shown as text. No separate diff-review step exists because there's no "before"
version to diff against. Read the content preview carefully before saying yes.

**Multiple files in one turn** → prompts come **one at a time**, one per file, not batched. For
risky/complex changes, approve each file individually (`1` each time). For low-risk, repetitive
changes across many files, picking `2` once (accept-edits) saves time but skips all further review
for the rest of the session.

### Checking usage / context

| Command (type in Claude prompt) | Shows |
|---|---|
| `/usage` | Plan quota: 5-hour window %, weekly window %, reset times (Pro/Max). API users see `/cost` instead — session $ spend. |
| `/context` | How full **this chat's** context window is (colored grid, per-category breakdown) |
| `/compact` | Summarize old history to free context space, keep working |
| `/clear` | Wipe context, start fresh chat (old one still resumable) |

**Persistent status line (no more typing `/usage` every time):** already set up today in
`~/.claude/settings.json` — shows Context %, 5h %, Week % permanently at the bottom of every
Claude Code session, auto-updating after each message.

### Debug spam fix (applied today)

`lua/plugins/claudecode.lua` had `log_level = "debug"` left on, which printed every MCP tool call
(`closeAllDiffTabs`, etc.) as a notify on every prompt. Changed to `log_level = "info"`. If you ever
see this again, that's the setting to check.

### Troubleshooting checklist if diffs/prompts misbehave

```
:checkhealth claudecode     " confirms CLI found, WS server running, lock file exists, client connected
:ClaudeCodeStatus           " connection status
:ClaudeCodeCloseAllDiffs    " clears stuck/phantom diff tabs
```
Also check the mode badge at the bottom of the Claude pane — most "why isn't it asking me" confusion
today traced back to being in `accept edits` mode, or a file already approved once this session.

---

## PART 2 — Full LazyVim Keymap Cheatsheet

Leader = `Space`. Press `Space` and wait — **which-key** popup shows everything live, this table is
just for offline revision.

### File search / navigation (what you said you forgot)

| Keys | What |
|---|---|
| `<leader><space>` | **Find files** (fuzzy, fastest) |
| `<leader>ff` | Find files (explicit) |
| `<leader>fr` | Recent files |
| `<leader>/` | **Live grep** — search text across whole project |
| `<leader>sg` | Grep (same idea, LazyVim's "search" group) |
| `<leader>ss` | LSP document symbols (functions/classes in current file) |
| `<leader>e` | Toggle file **explorer** sidebar (neo-tree/snacks) |
| `:NvimTreeToggle` | Alternative tree explorer (nvim-tree, this config's own) |

### Terminal

| Keys | What |
|---|---|
| `<leader>ft` or `Ctrl-/` | Open/toggle a terminal split |
| `<leader>tt` | Toggle the Maven-specific bottom terminal (this config's custom one) |

Terminal mode basics (any terminal buffer, incl. Claude's):
- You're in **terminal/insert mode** when typing goes to the running process
- `Esc` or `Ctrl-\ Ctrl-n` → drop to **Neovim normal mode** over the terminal buffer (to scroll/copy)
- `i` or `a` from there → back into terminal/insert mode (process receives keys again)

### Your custom keymaps

| Keys | Mode | What |
|---|---|---|
| `<leader>z` | n | Run current file (java/js/dart/py/go) in a float |
| `<leader>mc` | n | `mvn clean install` in bottom terminal |
| `<leader>mt` | n | `mvn test` |
| `<leader>mr` | n | `mvn compile exec:java` |
| `<leader>md` | n | `mvn dependency:resolve` |
| `<leader>tt` | n | Toggle/shrink Maven terminal |
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
| `<leader>ar` | n | Resume Claude Code (picker) |
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

Commands: `:OpenConfig`, `:OpenPlugin`, `:NvimTreeToggle`, `:Leet`, `:Maven`,
`:LiveServerInstall`, `:ClaudeCode`, `:AvanteToggle`, `:AvanteAsk`.

### LazyVim defaults you'll actually use

| Keys | What |
|---|---|
| `<leader><space>` | Find files |
| `<leader>/` | Grep (live) |
| `<leader>ff` | Files |
| `<leader>fr` | Recent files |
| `<leader>sg` | Grep |
| `<leader>ss` | LSP document symbols |
| `<leader>e` | Explorer |
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

## PART 3 — Quick daily-use cheat card

```
Space  <space>   → find file
Space  /         → grep text in project
Space  e         → file explorer
Ctrl+/           → terminal
Space  ac        → open Claude
Shift+Tab (in Claude, insert mode) → cycle permission mode
Space  aa / ad   → accept / deny Claude's diff
Space  ar        → resume a specific old Claude chat
/clear           → new Claude session (inside Claude)
/context         → check context window fill
/usage           → check plan quota
```

---

*Full official cheatsheet with more detail lives in `revise.md` — this file is just today's
condensed study notes.*
