# Evilify Skill v2 Redesign

## Why v1 was not enough

The original `evilify/SKILL.md` was 25 lines of high-level behavioral principles. When coding agents applied it to real desktop and web applications, those gaps showed up immediately in shipped interfaces:

- **No persistent modal status line:** agents fell back to fleeting toasts on mode transitions, leaving users blind to their active mode, focused pane, numeric count, and pending prefix.
- **No localleader model:** object-specific and view-specific commands were crammed into deep 4-stroke `SPC` trees instead of living on a context-scoped localleader (`,` and `SPC m`).
- **Thin UI specification:** without concrete geometry, visual hierarchy, and anti-patterns, agents rendered which-key inside plain toasts, dumped dozens of unaligned raw lines into `?`, and built cramped ASCII-text command palettes lacking `amx` recency ranking, `orderless` matching, input history, or right-aligned live keycaps.
- **Incomplete input and customization rules:** non-Latin physical key handling, macOS Option (`M-`) chords, `DEL` step-back inside prefixes, `:localleader` / `:keyless` EDN extensions, auto-created starter `keys.edn` files, and user-visible validation banners were absent or scattered across individual apps.

## What v2 keeps from v1

- One command registry driving execution, which-key, command search, cheatsheets, and describe sheets from the same effective keymap.
- Mnemonic-first, reach-second letter choice (home row, then bottom row, then top row).
- Developer documentation rule (linked from `AGENTS.md`) requiring every new page, panel, or action to ship with its command in the same change.
- Full backward compatibility with the portable EDN subset in [evilify/keymap-edn.md](../evilify/keymap-edn.md).

## What v2 redesigned from scratch

- **Modular skill layout:** `evilify/SKILL.md` stays concise and actionable while linking three focused reference specs under `evilify/references/` (`key-architecture.md`, `ui-surfaces.md`, `verification.md`) and a generic starter template under `evilify/templates/keys-template.edn`.
- **Mandatory modal status line:** specifies exact left-to-right zones, semantic mode and capture badges (`NORMAL`, `INSERT`, `VISUAL`, `LEADER`, `LOCAL`, `HINTS`, `M-x`, `KEY?`), reserved layout height, 0ms count/prefix echo, and touch-viewport hiding.
- **Complete leader, localleader, and mnemonic system:** defines Doom-aligned `SPC` groups, `,` and `SPC m` context maps with an empty-context guard, Vim prefixes (`g`, `y`, `z`, `C-w`), Shift semantics, safe short keys, numeric counts, and physical key resolution on non-Latin and macOS Option layouts.
- **Purpose-built visual surfaces:** separates the curated task-card `?` cheatsheet from the exhaustive `SPC h b` / `SPC ?` active binding list, docks which-key flush above the status line with `+group` vs leaf differentiation, anchors `M-x` / `SPC SPC` in the upper third with `amx` + `orderless` + `M-p`/`M-n` history and matched-character highlights, and defines explicit anti-patterns.
- **EDN optional extensions:** documents `:localleader`, `:keyless`, starter user files, and visible validation diagnostics as optional extensions so portable-subset parsers remain fully valid.

## Key tradeoffs

- **Separate `?` cheatsheet vs `SPC h b` live bindings:** maintaining two views adds a small curated card definition (referencing command IDs, never hardcoded keys), but prevents `?` from turning into an unreadable 100-row wall of text while keeping every active binding one `TAB` press away.
- **Upper-third anchored `M-x` vs vertically centered modal:** anchoring near the top leaves space below short result lists, which is far better than having the prompt input jump vertically on every keystroke as candidates narrow.
- **Optional EDN extensions vs expanding the portable subset:** keeping `:localleader`, `:keyless`, and prefix labels as documented optional extensions lets simpler clients stay strictly compatible with the portable subset without breaking existing files.
