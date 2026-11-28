# Verification Checklist

An evilified interface is not done when the bindings compile. Verify three layers before calling the work complete: keymap and coverage invariants, real runtime keyboard behavior, and visual craft across every user-visible surface in light and dark themes.

## 1. Automated Keymap and Coverage Checks

Run these in fast unit tests (headless, no manual clicking required):

- **EDN reader and validation suite:**
  - Accepts comments (`;`), discard forms (`#_`), string escapes, and valid overlays.
  - Rejects empty files, multiple top-level forms, tagged literals, unknown named maps, unknown command IDs, malformed chords, raw duplicates, normalized duplicates (`"C-x"` and `"Ctrl+x"`), command/prefix collisions within one map (`"g"` and `"g s"`), reserved chords taken, required bindings removed, and unsupported value types.
  - Confirms atomic fallback: any single error preserves 100% of built-in defaults and produces a diagnostic naming the offending map, chord, and line number.
  - Confirms the auto-generated starter `keys.edn` parses cleanly both as shipped (all examples commented) and with its commented examples enabled.
- **Effective map precedence tests:**
  - Per-map overlay runs before context precedence.
  - `nil` removes only that exact chord in that map, leaving extensions and other maps intact.
  - Higher-priority maps shadow both identical chords and prefix/command collisions in lower maps.
  - Prefix labels never create phantom prefixes and fail validation on leaf commands or empty prefixes.
- **Command and button coverage tests:**
  - Fails on any registered command that has no binding and is not listed under `:keyless`.
  - Fails on any interactive button missing both a command wiring (`data-cmd` or framework equivalent) and an explicit opt-out (`data-keys="none"`).
  - Remap a command and verify its button, accessible shortcut, help and dispatcher agree. Check focused activation on keyless buttons and reject unknown IDs.
- **`M-x` ranking and matching tests:**
  - Empty query orders top-N recent commands first, then frequency, then shorter labels, then alphabetical.
  - Multi-word `orderless` queries match in any order via plain substring, safe regex (invalid regex syntax does not throw), and word initialisms.
  - `M-p` / `M-n` step through input history and restore the current draft cleanly.

## 2. Real Runtime Interaction Checks

Exercise the actual application (browser automation, desktop UI harness, or real event dispatch) across these paths:

- **Mode transitions and escape ladder:** Normal -> Insert -> `ESC`/`C-g`/`C-[` leaves input selected (dashed outline) -> `i` returns to the exact input with cursor intact -> second `ESC` clears outline -> subsequent `ESC` steps back one layer at a time.
- **Leader and localleader lifecycle:**
  - `SPC` updates status line to `LEADER` at 0ms; pausing reveals docked which-key drawer; `DEL` pops one stroke (`SPC b` -> `SPC`); `ESC` cancels; unknown ending (`SPC b q`) shows undefined notice and does not fall through to run `q`.
  - `,` and `SPC m` update status line to `LOCAL` with active context name, open context continuations in which-key, and when no context map applies show a notice and swallow the next keystroke.
- **Counts and repeats:** verify `5j`, count echo and count limits. Hold a motion to scroll; hold Return over an action or confirmation and verify a single activation. Synthetic key events usually carry no autorepeat: test held-key timing with a fake clock in unit tests, and ask a human for one physical check.
- **Focus, pane, and window transitions:** switching pane, tab, account, dialog, or window cancels any pending prefix, count, or hints; active maps stay frozen across strokes of an in-flight chord.
- **IME, non-Latin layouts, and macOS `M-`:**
  - IME composition (`isComposing`) and native Insert typing work undisturbed.
  - Physical key positions execute Normal, leader, and hint bindings while a non-Latin layout (such as Persian, Arabic, or Russian) is active.
  - macOS `Option` (`M-x`, `M-p`, `M-n`) matches by physical key without leaking accented/symbol characters into focused inputs.
- **Chord vs palette vs mouse parity:** invoking a command via shortcut, via `M-x` / `SPC SPC`, and via mouse button calls the exact same handler with identical applicability checks, confirmations, and undo behavior.
- **Hints safety:** Avy hints skip scrolled-out or overlay-occluded targets, narrow cleanly as prefix letters are typed, and cancel cleanly if the target mutates or disappears before activation.
- **Filtering:** filter to zero, clear while a request is pending, and activate the highlighted result after a reorder. Verify its identity, focus and visible position. Test a synonym, a name in both keyboard layouts, and literal-only notes.
- **Mode-off fallback:** when modal operation is toggled off (in apps with a toggle), native application shortcuts and typing behave identically to stock.

## 3. Visual and Screenshot Audit

Inspect real rendered screenshots (never assume CSS or native layout looks right from code alone) in **both light and dark themes**, at desktop width and a narrower window width:

- **Modal status line:**
  - Verify `NORMAL`, `INSERT`, `VISUAL`, `LEADER`, `LOCAL` (with context chip), and pending prefix/count states read cleanly at a glance.
  - Confirm reserved bottom clearance: the last list row, detail text, and composer are never clipped or overlapped.
  - Confirm mobile/touch viewport hides the status line when no hardware keyboard is active.
- **Docked which-key drawer:**
  - Flush above the status line with zero gap or overlap; main app content remains visible above.
  - Columns align on a crisp vertical rail; keycaps never wrap or collide; `+group` entries visually stand out from leaf commands; long labels truncate cleanly without spilling out of columns.
- **Cheatsheet (`?`) and Live Binding Reference (`SPC h b`):**
  - Curated `?` cards look balanced and scannable (no giant scrolling wall of raw commands, live-resolved keycaps, clean footer showing user `keys.edn` path).
  - `TAB` switches cleanly to `SPC h b` grouped from most specific to most general, with no duplicate rows for multi-bound commands.
- **Describe Key (`SPC h k`) and Describe Command (`SPC h f`):**
  - `KEY?` badge is clear during capture; result cards have comfortable spacing and clearly distinguish available, unavailable-with-reason, and `:keyless` commands.
- **`M-x` and `SPC SPC` command palette:**
  - Anchored cleanly in the upper third and does not jump vertically as results filter from many rows to one.
  - Active row left accent bar, matched-character highlights, subtle secondary metadata, right-aligned live keycaps, and footer hint bar all have strong contrast and calm visual rhythm.
- **Avy hints, focus cues, and error banner:**
  - Hint tags remain legible over both light and dark backgrounds and highlight typed prefixes cleanly.
  - Active pane accent line, post-`Esc` dashed input outline, visible button shortcuts, and the invalid `keys.edn` banner are all crisp and unobtrusive.

Keep live-account fixtures and temporary inspection screenshots out of shared repositories; store scratch captures in your scratch directory.
