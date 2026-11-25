# evilify-skill

An agent skill for designing, implementing, and auditing Doom Emacs and Vim-style modal keyboard interfaces inside existing desktop, web, or terminal applications.

## What the skill covers

- **Modal architecture:** Normal, Insert, and Visual modes, transient capture states (`LEADER`, `LOCAL`, `HINTS`, `M-x`, `KEY?`), step-by-step `ESC` / `C-g` unwinding, numeric counts, physical key dispatch on non-Latin layouts and macOS Option (`M-`), and safe short keys.
- **Mnemonic leader and localleader design:** Doom-aligned `SPC` leader groups, context-scoped localleader (`,` and `SPC m`) with an empty-context guard, Vim prefixes (`g`, `y`, `z`, `C-w`), and the mnemonic-first, reach-second rule (home row, then bottom row, then top row).
- **User-visible surfaces:** a persistent modal status line (never toast-only feedback), a docked which-key drawer flush above the status line, four distinct help views (`?` curated task-card cheatsheet, `SPC h b` / `SPC ?` exhaustive live bindings, `SPC h k` describe key, `SPC h f` describe command), an upper-third `M-x` / `SPC SPC` command palette with `amx` recency/frequency ranking, `orderless` multi-word matching, and `M-p`/`M-n` input history, plus Avy hints, focus indicators, visible button shortcuts, and keymap error banners.
- **Shared EDN keymap format:** built-in default file plus user overlay, atomic validation, optional prefix labels, `:localleader`, `:keyless`, and an auto-created commented starter file.
- **Verification:** automated EDN, precedence, and command/button coverage tests, real runtime interaction checks, and rendered light/dark screenshot audits.

## Inside this repository

- [evilify/SKILL.md](evilify/SKILL.md): core workflow and non-negotiable rules.
- [evilify/references/key-architecture.md](evilify/references/key-architecture.md): modes, `SPC` leader taxonomy, localleader, mnemonic and reach rules, non-Latin physical keys, safety, and coverage.
- [evilify/references/ui-surfaces.md](evilify/references/ui-surfaces.md): complete visual and interaction spec for the status line, which-key drawer, cheatsheet, binding reference, describe cards, `M-x` palette, hints, and anti-patterns.
- [evilify/references/verification.md](evilify/references/verification.md): automated, runtime, and visual verification checklist.
- [evilify/keymap-edn.md](evilify/keymap-edn.md): shared EDN keymap specification and optional extensions.
- [evilify/templates/keys-template.edn](evilify/templates/keys-template.edn): generic starter user keymap template with teaching comments.
- [docs/redesign.md](docs/redesign.md): v2 redesign rationale, tradeoffs, and changes from v1.
