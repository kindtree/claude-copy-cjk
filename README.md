# claude-copy-cjk

Copy text from Claude Code's terminal UI and paste it clean — now with full CJK (中文/日本語/한국어) support.

A hard fork of [andersmyrmel/claude-copy](https://github.com/andersmyrmel/claude-copy) that fixes CJK text handling, cleans more Claude TUI artifacts, and works in more Claude Code hosts.

When you select and copy text from [Claude Code](https://claude.com/claude-code), your clipboard gets filled with rendering junk: 2-space margins, `⏺` / `❯` / `⎿` markers, spinner lines, box-drawing characters, and hard line breaks from terminal wrapping. This tool runs as a [Hammerspoon](https://www.hammerspoon.org/) key interceptor on macOS, catches `Cmd+C` in terminal apps, and cleans the clipboard before you paste.

## What this fork adds over upstream

**CJK correctness** (upstream heuristics measure bytes; CJK is 3 bytes but 2 display columns, which breaks every width-based heuristic):

- All wrap detection uses UTF-8 display width, not byte length — mixed 中英文 paragraphs rejoin correctly.
- Join separator is CJK-aware: a hard break inside 修改 rejoins as `修改`, never `修 改`.
- CJK sentence-final punctuation (。！？：；…) is recognized, so complete short sentences are not merged.
- Fullwidth-colon key-value lines (`标题：…`) keep their own lines.

**More artifacts cleaned:**

- `⏺` response markers, `❯` prompt echoes, and `⎿` tool-result markers are stripped.
- Spinner/status chrome (`✻ Worked for 11s`, `(ctrl+o to expand)`, `────` separators) is dropped.
- Common leading indent is removed from whole-line selections (relative/nested indent preserved).
- A partial 1-space margin on the first line (selection started mid-margin) is normalized.

**Reflow correctness:**

- Wrapped markdown list items (`- item` / `1. item`) rejoin into single lines — including greedy word-wrap leftovers like a short `3. API` head before a long CJK run.
- Column-aligned output (`ls`, `ps`, tables — runs of 3+ internal spaces) is never reflowed. Upstream merged `ls` rows into one line.
- Lua comments, calls with trailing comments, bare closing brackets, and `end)` are recognized as code, not prose.

**More hosts:** [cmux](https://github.com/manaflow-ai/cmux) and the Claude desktop app are whitelisted alongside Ghostty, iTerm2, Terminal, Alacritty, kitty, WezTerm, Hyper, Warp, Rio, Tabby, and Wave.

**Robustness:** clipboard race guard (won't clobber a newer copy), empty-result guard (never wipes the clipboard), oversized-content guard (>512 KB is passed through untouched), and a fast path that roughly halves large-clipboard processing time.

## Install

Requires macOS and [Homebrew](https://brew.sh/).

```bash
git clone https://github.com/kindtree/claude-copy-cjk.git
cd claude-copy-cjk
./install.sh
```

The install script installs Hammerspoon if missing, copies `init.lua` to `~/.hammerspoon/claude-copy.lua` and `clean.lua` to `~/.hammerspoon/clean.lua`, and appends a `dofile()` line to your `~/.hammerspoon/init.lua`. Then open Hammerspoon, grant Accessibility permissions, and reload the config.

## Configuration

Detection thresholds live at the top of `clean.lua`. This fork ships more aggressive defaults than upstream (clean even single-line copies):

| Key | This fork | Upstream | Meaning |
|-----|-----------|----------|---------|
| `minNonEmptyLines` | 1 | 2 | Minimum non-empty lines before cleaning is considered |
| `minMarginCoverage` | 0.50 | 0.65 | Fraction of lines that must carry the 2-space margin |
| `stripOnlyThreshold` | 2 | 4 | Score needed for margin-strip-only cleaning |

Terminal app whitelist lives at the top of `init.lua`.

## Tests

```bash
lua test.lua               # upstream suite (adapted to this fork's defaults)
lua test_deficiencies.lua  # CJK / TUI-artifact / reflow suite, built on real TUI captures
```

The `tui-fixture-*.txt` files are real Claude Code v2.1.207 renders captured via `tmux capture-pane`.

## Limitations

- macOS only (Hammerspoon requirement).
- Only plain keyboard `Cmd+C` is intercepted — copy-on-select and context-menu copy are not.
- Fenced code blocks are flattened by the terminal's own clipboard handling before this script runs.
- ASCII↔CJK join boundaries drop the space (`Python脚本`): word integrity is prioritized over spacing style, since the original spacing cannot be recovered from wrapped output.

## Credits

- Upstream: [andersmyrmel/claude-copy](https://github.com/andersmyrmel/claude-copy) (MIT), inspired by [Clean-Clode](https://github.com/TheJoWo/Clean-Clode).

## License

[MIT](LICENSE) — original copyright Anders, CJK and Claude-Code-host additions copyright kindtree.
