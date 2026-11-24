# Key Architecture and Modal Behavior

This reference defines the behavioral architecture behind an evilified application: modes, prefix resolution, the `SPC` leader taxonomy, localleader context maps, mnemonic and reach rules, non-Latin input handling, safety boundaries, and coverage guarantees.

## Modal State Machine

Keep the mode model small, explicit, and always visible on the modal status line.

### Primary modes

- **Normal** (default outside text inputs): keys dispatch commands.
- **Insert** (active when a text input, textarea, contenteditable region, or native dropdown has focus): plain keys and Shift enter text as usual. Only chords whose first stroke carries Control, Alt/Option, or Super (`M-x`, `C-[`, `C-g`, `M-'`) may intercept.
  - Leaving Insert via `ESC`, `C-g`, or `C-[` blurs the input back to Normal mode while keeping the input *selected* (marked with a subtle dashed outline). Pressing `i` in Normal mode returns directly to that selected box with the cursor intact; if no box is selected, `i` focuses the primary composer or input in view.
- **Visual** (for applications with selectable lists or collections): entered via `v` or `V`. Motion keys (`j`, `k`, `G`, `}`) extend the selection without triggering activation side effects such as marking items read or opening previews. Action keys (`d`, `p`, `x`) then act on the whole selection; `ESC` exits Visual and clears the selection.

### Transient capture states

While waiting for additional keystrokes, the dispatcher enters a transient capture state reflected immediately on the status line:

- **Leader (`LEADER`):** waiting for the next stroke after `SPC` (for example `SPC` -> `SPC b` -> `SPC b p`).
- **Localleader (`LOCAL`):** waiting for a context command after `,` or `SPC m`, scoped to the currently open object or page.
- **Prefix (`NORMAL` + pending echo):** waiting after a Vim prefix (`g`, `y`, `z`, `C-w`) or numeric count (`5`).
- **Hints (`HINTS`):** waiting for Avy-style target label characters (`f`, `y f`, `SPC j`).
- **Palette (`M-x` or `PICK`):** waiting for command or entity filtering in the command palette.
- **Describe-Key (`KEY?`):** capturing the next complete key sequence for `SPC h k` without executing it.

### Escape ladder (`ESC` and `C-g`)

Outside Insert mode, `ESC` and `C-g` unwind **exactly one layer per press**, in strict order:

1. Active transient capture or overlay (pending prefix, which-key, hints, palette, cheatsheet, describe sheet, modal dialog).
2. Visual mode.
3. Active multi-item selection.
4. Selected text-box outline left behind after exiting Insert mode.
5. Open detail panel or sub-page (returning to the list or parent view).
6. Active search query or filter.

Never collapse multiple layers in one `ESC` press. Users rely on predictable step-by-step backing out.

### Prefix lifecycle and focus safety

- **Freeze active maps at stroke 1:** determine the ordered stack of active named maps when the first stroke of a sequence arrives. A background update or focus shift mid-chord must not silently rebind the meaning of stroke 2.
- **Cancel on context change:** switching windows, tabs, accounts, modal dialogs, or panes cancels any pending prefix, count, or hint session.
- **Step back with `DEL` (Backspace):** inside a multi-stroke sequence (`SPC b`), pressing `DEL` pops the last stroke back to `SPC` (and updates the status line and which-key immediately). Pressing `DEL` on a single-stroke prefix cancels cleanly.
- **Never fall through on unknown endings:** if the user types `SPC b q` and no binding exists for `q` under `SPC b`, cancel the sequence and show `SPC b q is undefined` on the status line. Never pass `q` through to Normal mode where it might close the window.
- **Explain unavailable commands:** if a bound chord resolves to a command whose `when` predicate is false, show its `need` message on the status line (for example `Needs an open task`) instead of doing nothing silently.

## Eight Mnemonic and Keymap Design Rules

Derive every binding systematically rather than adding keys ad hoc as features appear.

### 1. Canonical group home plus short keys

Every command belongs in one discoverable group: a `SPC` leader group, the localleader context map, or a standard Vim prefix (`g`, `y`, `z`, `C-w`). Because every command has a group home, the which-key trees form a complete map of the application. Frequent commands also receive a single-key shortcut in Normal mode on top of their group home.

### 2. Doom-aligned `SPC` leader taxonomy

Keep top-level `SPC` letters aligned with Doom Emacs so existing habits transfer immediately. Adopt only the groups that fit the host application, and never repurpose a standard Doom letter to mean something unrelated:

- `SPC SPC`: universal quick finder (applicable commands plus app entities such as pages, chats, projects, or items; opens `M-x` directly in apps without entities).
- `SPC :`: `M-x` commands only (same as `M-x` and `:`).
- `SPC ,`: switch active item, chat, or buffer in the current view.
- `SPC ?`: all active bindings (alias for `SPC h b`).
- `SPC b`: buffer or active item (the primary open object: answer, pin, mark read, close, export, next/prev).
- `SPC d`: domain node or scope (when `p` is a frequent reach; `d` sits on the home row).
- `SPC f`: file operations (open, save, export, reveal).
- `SPC g`: version control or history (where relevant).
- `SPC h`: help (`h b` all active bindings, `h k` describe key, `h f` describe command).
- `SPC i`: insert or message box (focus input, attach file, voice input, formatting, send options, discard draft).
- `SPC j`: jump and hints (`SPC j` or `SPC j j` hints).
- `SPC m`: localleader alias (mirrors `,`).
- `SPC n`: new or create (new task, new message, new group, new item).
- `SPC o`: open a page, panel, or drawer (settings, logs, recordings, secondary views).
- `SPC p`: project or workspace switching and actions.
- `SPC q`: quit, session, reload, or lock actions.
- `SPC s`: search (`s s` search, `s c` clear search, filters).
- `SPC t`: toggles (view modes, sorting, visibility, theme, feature switches).
- `SPC w`: windows and panes (`w h`/`j`/`k`/`l` move focus, `w w` cycle, `w s`/`w d` focus specific panes).
- `SPC x`: selection and bulk list operations (`x a` select all, `x c` clear, `x d` bulk mark read).

### 3. Mnemonic first, then reach

Pick letters in this strict priority order:

1. **Established editor/Doom habit:** keep Doom group letters (`SPC b`, `SPC o`, `SPC w`, `SPC h`, `SPC t`) and Vim motion/prefix letters (`h j k l`, `g`, `y`, `z`, `C-w`, `/`, `?`).
2. **Short-key reuse under the leader:** if a command has a short key, reuse that exact letter inside its leader group (`p` pins, so `SPC b p` pins; `d` marks read, so `SPC b d` marks read; `x` selects, so `SPC b x` selects). Users learn a letter once.
3. **Ergonomic reach tie-breaker:** when two mnemonic letters are equally natural, prefer the **home row** (`a s d f g h j k l`), then the **bottom row** (`z x c v b n m`), then the **top row** (`q w e r t y u i o p`). Reserve easy home-row letters for daily groups (for example `SPC d` for a heavily used node/scope group rather than stretching to `SPC p` hundreds of times a day).

### 4. Localleader (`,` and `SPC m`) for context-only actions

Commands that make sense only for one kind of open object or page (a task, a deliverable, a media player, a failed message, a settings page, a log view) belong in a dedicated context keymap behind the localleader, not crowded into global `SPC`.

- Bind both `,` (single home-adjacent keystroke) and `SPC m` (Doom default) via `:localleader ["," "SPC m"]` in the EDN map so users can customize the localleader in one line.
- Reuse letters for the same semantic intent across different context maps:
  - `o` open or reopen,
  - `d` / `D` done or primary completion / destructive drop,
  - `r` retry, replace, release, or render,
  - `e` / `E` export (standard / full archive),
  - `y` / `n` approve (yes) / deny (no),
  - `z` / `Z` defer or snooze (standard / alternate scope),
  - `p` toggle primary setting, `t` test.
- **Empty-context guard:** when the current view has no active context map, pressing `,` or `SPC m` must display a calm status-line message (`No local keys here`) and swallow the next keystroke rather than falling back to Normal mode.

### 5. Vim prefixes keep their Vim meaning

- `g` (go): `g g` top/first, `g h` home/root, `g r` refresh, `g R` full reload, `g s`/`g S` local/global settings, `g o` open parent/owner.
- `y` (yank/copy): every copyable artifact lives under `y` (`y y` link, `y i` ID, `y p` path, `y f` copy visible element by hint).
- `z` (fold): `z a` toggle fold, `z c` close fold, `z o` open fold, `z M` fold all, `z R` unfold all.
- `C-w` (window/pane): `C-w h`/`j`/`k`/`l` directional focus, `C-w w` next pane.

### 6. Shift reverses direction or widens scope

Use Shift consistently so users can guess the shifted variant without checking help:

- Reverse direction: `n`/`N` next/previous open item, `}`/`{` next/previous section, `Tab`/`S-Tab` forward/backward.
- Wider scope or stronger variant: `g v`/`g V` local/global view, `g s`/`g S` view/global settings, `, z`/`, Z` defer/defer-for-quota, `, e`/`, E` single export/full archive.
- Cross-pane stepping: `j`/`k` move inside the focused pane; `J`/`K` (and `C-j`/`C-k`) step through the main list from any pane.

### 7. Short keys never commit irreversible actions

Single-key Normal bindings must be fast and safe:

- Moving, scrolling, opening previews, folding, pinning, toggling selection, and entering a composer are appropriate for single keys.
- Destructive or externally visible commitments (deleting, archiving an agent or chat, dropping a task, interrupting a job, sending a message, marking an entire workspace read) belong under the leader or localleader and keep the application's normal confirmation or undo protection.
- When moving a cursor in a list has side effects (such as marking items read on open), separate preview/selection movement (Visual mode `V` + `j`/`k`) from activation so users can inspect or batch-select without mutating read state.

### 8. Numeric counts

Digits `1` through `9` (plus `0` once a count has started) accumulate a numeric prefix shown live on the status line (`5j`, `3C-d`, `2}`).

- Apply counts only to commands marked repeatable (motions, scrolling, stepping). Ignore or reject counts on non-repeatable actions rather than running a destructive action 5 times.
- Cap accumulated counts (for example at 999 or the list length) to prevent UI freezes.
- Reserve bare leading digits in `:normal` so user keymaps cannot shadow counts.

## Physical Keys, Non-Latin Layouts, and Platform Shortcuts

Modal interfaces break badly for multilingual and macOS users if key normalization reads only translated characters.

- **Non-Latin keyboard layouts:** in Normal, Visual, Leader, Localleader, Prefix, and Hints modes, resolve letter and punctuation commands from **physical key positions** (such as `KeyboardEvent.code` on the web or OS scan codes / virtual keycodes on desktop) rather than layout-translated glyphs. A user typing with Persian, Arabic, Russian, or Greek input active can still press physical `j`, `k`, `SPC b p`, or `?` seamlessly.
- **Insert mode and IME composition:** never intercept plain characters while an input has focus or while IME composition (`isComposing` / dead-key composition) is active.
- **macOS Option (`M-`) chords:** macOS Option transforms letters into symbols (`Option-x` -> `≈`, `Option-p` -> `π`). Match `M-` chords by physical key (`Alt` + `KeyX` -> `M-x`, `Alt` + `KeyP` -> `M-p`), and call `preventDefault()` when a `M-` binding matches so stray symbols are never inserted into inputs.
- **Platform shortcuts:** preserve native OS and browser bindings (`Cmd+C`, `Cmd+V`, `Cmd+X`, `Cmd+A`, `Cmd+Z`, `Cmd+Q`, `Cmd+W`, `Cmd+Tab`, `Cmd+L`) unless the application explicitly owns a safe in-app override.

## Command Registry, Keyless Commands, and Coverage

Keep commands and bindings strictly decoupled and continuously verified:

- **One definition per command:** each command declares `id`, `label`, `description`, `when`, `need`, `repeatable`, `palette` (default true), and `run`.
- **Single-path execution:** a command executed via chord, via `M-x`, or via clicking its native button must run the exact same handler with the same guards, confirmations, and undo behavior. Tag buttons with their command ID (`data-cmd="pin"`) where the framework allows, so clicks, keys, and tooltips share one wiring.
- **Deliberately keyless commands:** commands reached only by name (`M-x`), by a cycling command, or by Avy hints are listed explicitly under `:keyless` in the EDN file (and row/dialog buttons tagged with an opt-out such as `data-keys="none"`). `SPC h f` displays them as `No key (on purpose)`.
- **Developer docs and `AGENTS.md` rule:** add a rule to the host repository's developer docs and link it from `AGENTS.md`: *every new page, panel, or user action must add its command (and key or `:keyless` entry) in the same change*. Back that rule with an automated coverage test so mouse-only regressions fail CI immediately.
