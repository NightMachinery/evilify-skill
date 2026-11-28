---
name: evilify
description: Add Doom Emacs or Vim-style modal keyboard operation to an existing application with a modal status line, SPC leader, localleader, mnemonic keymap in EDN, docked which-key drawer, cheatsheet, M-x command palette, and Avy-style hints. Use when designing, implementing, redesigning, or auditing a keyboard interface across app actions.
---

# Evilify an existing application

Give an existing application a cohesive, keyboard-first modal interface in the spirit of Doom Emacs and Vim. Use the application's native action handlers, focus APIs, and theme tokens so keyboard operation feels built in rather than bolted on.

Read the focused references before designing or implementing:

- [references/key-architecture.md](references/key-architecture.md): modal state machine, `SPC` leader taxonomy, localleader (`,` and `SPC m`), mnemonic and reach rules, Vim prefixes, non-Latin physical keys, safety, and coverage.
- [references/ui-surfaces.md](references/ui-surfaces.md): visual and interaction specification for the modal status line, docked which-key drawer, cheatsheet, live binding reference, describe sheets, `M-x` / `SPC SPC` command palette, Avy hints, focus cues, and anti-patterns.
- [keymap-edn.md](keymap-edn.md): shared EDN keymap format, portable subset, overlay and precedence rules, validation, and optional extensions (`:localleader`, `:keyless`, prefix labels).
- [references/verification.md](references/verification.md): automated keymap and coverage checks, runtime keyboard tests, and visual screenshot verification in light and dark themes.
- [templates/keys-template.edn](templates/keys-template.edn): generic starter user keymap template with teaching comments.

## 1. Audit the host application

Before binding a single key, map the application as it actually works:

- List its panes, views, text inputs, modal dialogs, platform shortcuts, and every action users currently reach by mouse.
- Inspect its theme tokens (surfaces, borders, primary and muted text, accent color) in both light and dark modes.
- Read any user keymap references within authorized scope and preserve exact keys the user requested.

## 2. Build one command registry

Give each action a stable ID, label, search aliases in the user's vocabulary, description, applicability test, disabled reason and native handler. Bind IDs in EDN. Dispatch, help, menus and visible button shortcuts all use the effective keymap. Where a button has no dedicated chord, show its actual focused activation key. Expose the same shortcut and reason to assistive technology.

## 3. Design the keymap

Follow [references/key-architecture.md](references/key-architecture.md) in full. The core rules:

- **Explicit modes:** Normal (commands), Insert (text inputs and native controls; only Ctrl/Alt/Super chords intercept), Visual/Select where range selection makes sense, plus transient capture states (Leader, Localleader, Hints, Palette, Describe-Key). `ESC` and `C-g` unwind one layer at a time; leaving a text box keeps it selected (dashed outline) so `i` returns to it.
- **Canonical home plus short keys:** give every command a home under a Doom-aligned `SPC` leader group (`b` item/buffer, `d` node/domain, `f` file, `h` help, `i` insert/composer, `n` new, `o` open page, `p` project, `q` session/quit, `s` search, `t` toggle, `w` window/pane, `x` select), a Vim prefix (`g` go, `y` yank, `z` fold, `C-w` panes), or the localleader. Frequent actions also get a single-key shortcut that reuses the same letter (`p` pins, so `SPC b p` pins).
- **Localleader for object-specific actions:** bind `,` and `SPC m` to context keymaps holding commands that belong only to the active object or page (such as a task, media item, or settings view). Reuse the same letters for the same ideas across contexts. When no context map applies, show a status-line notice and swallow the next keystroke.
- **Mnemonic first, then reach:** keep established Doom group letters and reused short keys first; among equally mnemonic letters, prefer the home row, then the bottom row, then the top row.
- **Safe short keys and Shift discipline:** Shift reverses direction or widens scope (`n`/`N`, `g v`/`g V`). Short keys never commit irreversible actions (delete, archive, drop, send); those live under leader/localleader with their normal confirmations. Held motions repeat smoothly. Return invokes the visible default once, like clicking the item. Moving the cursor never activates it.
- **Physical keys and IME safety:** dispatch Normal, prefix, and hint keys by physical key position so non-Latin layouts work seamlessly, and read physical keys for macOS `M-` (Option) chords. Leave Insert typing, IME composition, and established platform shortcuts intact.
- **Prefix ergonomics:** freeze active maps on the first stroke, cancel pending sequences on focus or mode change, let `DEL` back up one stroke inside a prefix, never fall through on unknown endings, and report `need` when an unavailable command is invoked.
- **One list-motion handler:** every list, picker and palette takes its motions (`j`/`k`, half and full page, first/last, held repeat) from one shared handler, so a new motion works everywhere and a new list inherits them. A list that already uses a chord (drag-reorder) keeps it and shows that in help.
- **Coverage and developer rule:** mark intentionally unbound commands under `:keyless` and row/dialog buttons with an explicit opt-out. Write into the app's developer docs (and link from its `AGENTS.md`) that every new page, panel, or action gets a command in the same change.

## 4. Build every user-visible surface with visual polish

Follow [references/ui-surfaces.md](references/ui-surfaces.md). Use host theme tokens, scrollbars and consistent keycaps. Allow room for names, actions and shortcuts. Reflow at narrow widths and retain the active object on resize or fullscreen.

- **Modal status line (required on desktop/fine-pointer):** a persistent bar joined to the active content, with its own layout space. Shows a semantic mode/capture badge (`NORMAL`, `INSERT`, `VISUAL`, `LEADER`, `LOCAL`, `HINTS`, `M-x`, `KEY?`), active pane and object context, 0ms pending count and prefix echo (`5`, `SPC b -`), transient feedback slot, and right-aligned selection/position/scope/help cues. Hidden by default on touch-only screens without a hardware keyboard.
- **Docked which-key drawer:** flush above the status line (never a centered blocking modal), appearing after a short pause (250 to 350ms) or immediately on explicit help. Shows a breadcrumb header (`SPC b` · group name, `DEL` back, `ESC` cancel) and a balanced multi-column grid of active effective continuations, visually distinguishing `+group` prefixes from human-labeled leaf commands.
- **Four distinct help surfaces:**
  - `?` (Cheatsheet): a curated, task-organized 2-to-3 column card layout (6 to 8 cards, 5 to 9 rows each) with live-resolved keycaps, `TAB` to switch to all bindings, and a footer naming the user's `keys.edn` path and status.
  - `SPC h b` / `SPC ?` (All Active Bindings): exhaustive, scrollable/filterable list of every binding active right now, grouped from most specific to most general (Localleader, Focused Pane, Leader groups, Everywhere), with `TAB` back to `?`.
  - `SPC h k` (Describe Key): captures any chord without executing (`KEY?` status badge) and reports what it runs here plus where else it is bound.
  - `SPC h f` (Describe Command): inspects any command (including unavailable and `:keyless` ones), showing applicability, why it is unavailable if so, and all bindings or "no key (on purpose)".
- **`M-x` and `SPC SPC` command palette:** anchored in the upper third of the viewport (so the input never jumps vertically) with prompt badge, borderless input, live `shown/total` counter, `amx` recency/frequency ranking, `orderless` multi-word matching (plain text, safe regex, initialisms), `M-p`/`M-n` input history, matched-character highlighting, soft left-accent active row, and right-aligned live keycap badges. `M-x` (`:`, `SPC :`) searches commands only; `SPC SPC` searches commands and app entities (or opens `M-x` directly). Closing restores the invoking focus. A filterable list offers a visible filter entry key; its first Escape keeps the filter and focuses the results, the next clears it. Clearing restores the source order and the cursor by identity, including after late results.
- **Hints and list state.** Revalidate each visible hint target on activation. Give cursor, marks, active item, hover and unavailable items distinct cues that can coexist. Expose their meanings to assistive technology.
- **Feedback.** Keep actionable errors and recovery in the surface that owns the work. Distinguish loading, empty, no matches, waiting to retry and failure. For saved or shared changes, report only the local or remote state actually observed.

## 5. Support local keymap customization

When the app allows remapping, ship the built-in map and load a user overlay in the shared EDN format ([keymap-edn.md](keymap-edn.md)) using a maintained non-evaluating EDN parser. Validate the user file atomically: on any error, keep built-in defaults and surface the offending map, chord, and line number in a visible banner. Auto-create a commented starter file (modeled on [templates/keys-template.edn](templates/keys-template.edn)) when missing, without touching existing files.

## 6. Verify behavior and visual craft

Use [references/verification.md](references/verification.md). Run automated EDN validation, command/button coverage tests, and real interaction tests (focus transitions, `ESC` unwinding, counts, incomplete sequences, IME and non-Latin layouts, macOS `M-` keys, stale/occluded hints, chord vs palette parity, and mode-off behavior). Then render and visually inspect screenshots of every surface (status line, which-key, cheatsheet, binding reference, describe sheets, `M-x`, hints, and error banner) in both light and dark themes before calling the work done.
