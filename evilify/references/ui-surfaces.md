# Visual and Interaction Design of Modal Surfaces

Every user-visible surface of an evilified application must look intentional, calm, legible, and native to the host application. Keyboard overlays fail when they look like raw debug panels bolted onto the UI, or when they disappear into fleeting toasts.

This reference specifies the visual foundation and the exact layout, typography, states, and ergonomics of every surface: the modal status line, the docked which-key drawer, the four help views (cheatsheet, live binding reference, describe-key, describe-command), the `M-x` / `SPC SPC` command palette, Avy hints, focus cues, visible button shortcuts, and validation banners.

## Visual Design Foundation

### Host theme inheritance and light/dark parity

Never hardcode a dark overlay onto a light app (or bright primary colors that clash with the host chrome). Map every modal surface to a small set of semantic tokens inherited from the host theme:

- `--ev-surface`: elevated overlay background (which-key, palette, cheatsheet, describe cards).
- `--ev-sunken`: recessed fill for the status line, keycap interiors, and subtle badges.
- `--ev-hover`: soft tinted row background for active candidates in `M-x` and lists.
- `--ev-border` and `--ev-border-strong`: 1px structural hairlines and crisp keycap/overlay outlines.
- `--ev-text` and `--ev-muted`: primary ink and calm secondary metadata.
- `--ev-accent` and `--ev-accent-soft`: primary accent ink (for `+group` prefixes, active left bars, matched characters) and its 10 to 15 percent translucent tint.
- `--ev-shadow`: soft multi-layered elevation shadow (tight ambient ring plus deeper downward diffuse shadow; upward shadow for the docked which-key drawer).

Verify WCAG AA contrast in both light and dark modes: at least 4.5:1 for all labels, descriptions, and keycaps, and at least 3:1 for hairlines and mode badges.

### Typography

- **UI sans-serif (host font):** headings, command labels, descriptions, and help prose.
- **Monospace (`tabular-nums`):** keycaps, chord breadcrumbs, numeric counts, command IDs, list positions (`4/19`), and palette match counters (`12/84`). Tabular figures prevent horizontal jitter as counters update.
- **Scale and hierarchy:**
  - Micro-labels (mode badge, card headings, scope headers, footer cues): `11px`, semibold/bold, subtle uppercase tracking (`0.04em` to `0.06em`) where uppercase is used.
  - Body rows (which-key items, cheatsheet rows, binding rows, `M-x` labels): `12.5px` to `13.5px`, generous line height (`1.4`).
  - Palette prompt input: `15px` to `16px`, clean weight (`400` to `450`), comfortable breathing room.

### Keycap (`<kbd>`) craftsmanship

Keycaps are the signature visual unit across every surface. Render them consistently everywhere:

- Monospace `11px`, medium-semibold (`500` to `600`), `line-height: 1.45`, `white-space: nowrap`.
- Subtle tactile cap geometry: `padding: 1px 6px`, `min-width: 20px` (centered so single letters like `j` and `k` feel balanced), `border-radius: 5px`, `1px solid var(--ev-border-strong)`, and a delicate `0 1px 0 var(--ev-border-strong)` bottom micro-bevel on `--ev-sunken` (or slightly lifted `--ev-surface` inside recessed containers).
- **Multi-stroke chords:** render each stroke as its own spaced keycap (`SPC` `b` `p`) with a `3px` to `4px` gap so chords scan rhythmically instead of blurring together.
- **Alternative keys:** separate equivalent keys with a quiet muted `/` (`j` / `k`), never cramming both into one keycap.

### Semantic mode color palette

Pick muted, authoritative colors with crisp white (or high-contrast) badge text so each modal state reads peripherally without screaming:

- **NORMAL:** forest green (for example `#2f7d44` light, `#358c4d` dark). Calm resting baseline.
- **INSERT:** cobalt blue (for example `#2563eb` light, `#3b82f6` dark). Clear text-entry signal.
- **VISUAL:** warm amber/ochre (for example `#b45309` light, `#d97706` dark). Active selection.
- **LEADER (`SPC`):** indigo/violet (for example `#6d28d9` light, `#7c3aed` dark). Global prefix capture.
- **LOCAL (`,` or `SPC m`):** deep teal (for example `#0f766e` light, `#0d9488` dark). Instantly distinct from global `LEADER` so the user knows they are inside an object's localleader.
- **HINTS / M-x / PICK / KEY?:** plum or violet accent matching interactive capture.

### Elevation stack and motion

Keep a strict z-index ladder so surfaces never collide:

1. Application content.
2. Focused pane accent border and post-`Esc` selected input outline.
3. Persistent modal status line (bottom edge).
4. Docked which-key drawer (flush directly above the status line).
5. Scrimmed centered/upper-third overlays (`M-x` / `SPC SPC`, `?` cheatsheet, `SPC h b` bindings, `SPC h k` / `SPC h f` describe sheets).
6. Avy hint tags (topmost during hint capture) and top keymap-error banner.

Motion must feel instantaneous:

- **0ms latency** on mode badge updates, count/prefix echo on the status line, and `M-x` filtering.
- **60ms to 90ms** subtle ease-out on overlay entrance (disabled when `prefers-reduced-motion: reduce` is set).
- **250ms to 350ms** idle reveal delay on the which-key drawer so fast muscle-memory chords (`SPC b p`) never flicker the bottom of the screen while the status line echoes each keystroke immediately.

## Surface 1: Modal Status Line (Mode Line)

### Why it is mandatory

Fleeting toasts cannot replace a status line. Without a persistent status bar, users cannot tell at a glance whether they are in Normal or Insert mode, whether `SPC` or `,` registered, which pane owns `j`/`k`, or whether a numeric count is active.

### Placement and responsiveness

- Docked flush to the bottom edge of the application viewport (`height: 26px` to `28px`), full width, with a crisp `1px` top border (`var(--ev-border)`).
- **Reserve its height in the app layout** (flex column or bottom padding equal to the bar's height) so it never hides the last row of a list, the bottom of a conversation, or a composer's send button.
- **Touch / mobile viewports:** hide by default on coarse-pointer touch screens (`(hover: none) and (pointer: coarse)`) so phone layouts keep their space; reveal automatically if a physical keyboard event arrives.

### Left-to-right anatomy

Keep zones in a fixed left-to-right order so the user's eye always knows where to look:

1. **Mode / capture badge** (left-flush, full height of the bar, `padding: 0 11px`, bold `11px` uppercase monospace, semantic mode fill):
   - Resting: `NORMAL`, `INSERT`, `VISUAL`.
   - Active capture: `LEADER`, `LOCAL` (or `LOCAL · TASK` when a context map is active), `HINTS`, `M-x`, `PICK`, `KEY?`.
2. **Active pane and context chip:**
   - Semibold primary text naming the focused pane (`sidebar`, `list`, `detail`, `dialog`).
   - When an object or page with a localleader map is active, show a subtle recessed context pill right beside the pane (for example `task`, `deliverable`, `settings`). This quietly teaches the user that `,` has actions here even before they press it.
3. **Live count and pending prefix echo (0ms):**
   - Shown whenever a count or incomplete sequence is active.
   - Rendered as a crisp accent-tinted monospace chip: `5`, `SPC b -`, `, -`, `g -`, optionally followed by the prefix's muted group label (`+item`).
4. **Transient feedback slot:**
   - Single-line status feedback (`Copied link`, `SPC b q is undefined`, `Needs an open task`, `No local keys here`). Truncates with ellipsis if space is tight; clears on the next command or after ~3 seconds.
5. **Flexible spacer.**
6. **Right metadata cluster** (muted `11.5px`, `tabular-nums` monospace for numbers, drops lower-priority items cleanly on narrow viewports):
   - Active selection count pill in accent tint when items are selected (`3 selected`).
   - Current list position (`4/19`) when in a list.
   - Current view/scope and active search query (`inbox /urgent`, max-width capped with ellipsis).
   - Quiet discovery cue on wide screens (`SPC` leader · `,` local · `?` help), lowest priority so it hides first when narrow.

## Surface 2: Docked Which-Key Drawer

### Geometry and placement

- Docked **flush directly above the modal status line** (`bottom: var(--ev-modeline-h)`), spanning the viewport width (or centered up to `1280px` on ultrawide displays with top-left/top-right `10px` radius), with a crisp `1px` top border and soft upward shadow.
- Never float which-key as a centered modal box that covers the user's active reading area.
- Cap maximum height at `38vh` so the main application remains clearly visible above it.

### Header strip

A compact single row (`padding: 8px 16px`, hairline bottom border) containing:

- **Left:** the pending chord rendered as individual `<kbd>` keycaps (`SPC` `b`), followed by the prefix's human label (`item`), and a subtle teal `localleader · task` pill when entered via `,` or `SPC m`.
- **Right:** quiet micro-controls (`DEL` back · `ESC` cancel, plus `C-n` / `C-p` and `Page 1/2` when paginated).

### Multi-column continuation grid

- Responsive CSS grid / multi-column layout (`grid-template-columns: repeat(auto-fill, minmax(215px, 1fr))`, `gap: 5px 20px`, `padding: 10px 16px 12px`).
- **Row anatomy:**
  - Fixed-width key slot (`32px` to `38px`, right-aligned or centered `<kbd>` so every label starts on a clean vertical rail).
  - Clean breathing gap (or subtle muted `→` arrow with low contrast, never heavy ASCII punctuation).
  - Concise human label (`1` to `4` words, single line with ellipsis only as last resort). Never display raw internal IDs like `open-archived-chats`.
- **Visual distinction between groups and leaves:**
  - Prefix groups carry a leading `+` added by the renderer (not duplicated inside the EDN label), styled in `var(--ev-accent)` with semibold weight (`+item`, `+open`, `+toggle`).
  - Leaf commands render in primary `var(--ev-text)` at regular weight.
- **Ordering:** sort consistently within the grid: either prefix groups first (alphabetically) then leaf commands (alphabetically), or case-folded alphabetical with lowercase/uppercase pairs (`d` then `D`) kept adjacent.
- **Applicability filtering:** show only continuations that apply in the current context (`when` is true) so every displayed key actually works.
- **Empty localleader handling:** if `,` or `SPC m` is pressed when no context bindings apply, do not open an empty drawer; display `No local keys here` in the status line and swallow the next keystroke.

## Surface 3: Help Views (`?`, `SPC h b`, `SPC h k`, `SPC h f`)

Never collapse all help into one giant scrolling dump. Separate **learning** (`?`), **exhaustive lookup** (`SPC h b`), and **inspection** (`SPC h k`, `SPC h f`) into four crisp, purpose-built surfaces.

### 3A. Curated Cheatsheet (`?`)

- **Purpose:** teach the entire modal mental model on one screen in 15 seconds.
- **Geometry:** centered modal (`width: min(1000px, calc(100vw - 32px))`, `max-height: 84vh`), soft scrim backdrop, `14px` corner radius, crisp `1px` border, flex column with sticky header, scrollable body, and sticky footer.
- **Header:**
  - Left: title (`Keyboard Cheatsheet`) and a short live subtitle (`Normal mode · live bindings`).
  - Right: segmented tab cue or action chip (`TAB` All active bindings) and `q` / `ESC` close.
- **Body (task-oriented card grid):**
  - 6 to 8 curated cards arranged in a balanced 2-to-3 column responsive grid (`repeat(auto-fill, minmax(280px, 1fr))`, `gap: 12px`, `padding: 16px 20px`).
  - Example card groups: *Navigate*, *Active Item*, *Leader (`SPC`)*, *Localleader (`,`)*, *Copy & Go*, *Panes & Folds*, *Selection, Hints & Typing*.
  - Each card has a subtle surface contrast (`1px solid var(--ev-border)`, `10px` radius, `12px 14px` padding), an uppercase/semibold accent heading with a faint bottom hairline, and **5 to 9 rows**.
  - **Row layout:** fixed two-column grid (`grid-template-columns: 104px 1fr`, `gap: 10px`, `padding: 3.5px 0`). Keycaps align cleanly on the left; concise descriptions align on a crisp rail on the right.
  - **Live key resolution:** cheatsheet rows reference command IDs (and the dynamic localleader token), never hardcoded strings. User remaps appear automatically; unbound commands drop cleanly from the card.
- **Footer:**
  - Hairline top border, single calm muted line: explains `ESC` step-by-step backing out, names the user's `keys.edn` path (`~/.config/<app>/keys.edn`), and indicates whether the user file is active or in fallback due to an error.

### 3B. Exhaustive Live Binding Reference (`SPC h b` and `SPC ?`)

- **Purpose:** inspect *every* binding currently active in the focused pane and context.
- **Geometry:** centered modal (`width: min(780px, calc(100vw - 32px))`, `max-height: 84vh`) with sticky header, an optional live filter bar (`/` focuses filter, `j`/`k` scrolls when unfocused), and `TAB` to switch back to the curated `?` cheatsheet.
- **Scope ordering (most specific first):**
  1. Active localleader map (`Localleader · Task (, / SPC m)`).
  2. Focused pane (`In the list`).
  3. Leader groups (`SPC b · item`, `SPC o · open`, ...).
  4. Everywhere (`Normal`) and text-box chords (`Insert`).
- **Row layout:** group multiple bindings for the same command onto a single row (`SPC b p` / `p` -> `Pin or unpin` `:pin`) so the list stays scannable and non-repetitive. Show subtle muted `:command-id` on the right for easy copying into `keys.edn`.

### 3C. Describe Key (`SPC h k`)

- **Capture phase:** status line badge switches immediately to `KEY?` with prompt `Press a key sequence to describe (C-g to cancel)`. Multi-stroke sequences (`SPC b p`, `, d`, `g g`) capture completely without invoking the command.
- **Inspection card** (`width: min(520px, calc(100vw - 32px))`):
  - Header showing the captured chord in prominent `<kbd>` caps (`SPC` `b` `p`).
  - Primary result: clearly names the command label, monospace `:command-id`, and brief description in the current context (or states `Unbound in this context`), with an interactive `Describe command` button/Enter shortcut to jump into `SPC h f` for that command.
  - Secondary `Elsewhere` list showing where the same chord is bound in other panes, contexts, or modes.

### 3D. Describe Command (`SPC h f`)

- Opens the command picker over **all** registered commands (including currently unavailable and `:keyless` commands).
- **Inspection card** (`width: min(520px, calc(100vw - 32px))`):
  - Command title, monospace `:command-id`, and concise description.
  - Status pill: `Available here` (green soft tint) beside its current keycap(s), or `Unavailable here` (amber/muted tint) beside its explicit `need` reason (`Needs an open task`).
  - `All bindings` list naming every chord and the scope where it works (for example `p` everywhere, `, d` with a task open), or `No key (on purpose) · run via M-x` for `:keyless` commands.

## Surface 4: `M-x` and `SPC SPC` Command Palette

The command palette is the power user's safety net and primary teaching tool. It must feel fast, uncluttered, and crisp on every keystroke.

### Role distinction

- **`M-x` (also `:` and `SPC :`):** commands only. Lists every registered command (including `:keyless` commands). Applicable commands rank first; unavailable commands either stay searchable with a clear muted reason pill (`needs open item`) or appear in a separate section below, never executing silently.
- **`SPC SPC`:** universal quick finder combining applicable commands with first-class application entities (pages, projects, chats, items), each clearly badged by kind (`Command`, `Page`, `Item`). In apps without searchable entities, `SPC SPC` opens `M-x` directly.

### Geometry and anchoring

- **Anchor in the upper third** of the viewport (`padding-top: 11vh`, centered horizontally, `width: min(640px, calc(100vw - 32px))`). Never vertically center the palette: as results shrink from 12 rows to 1 row, a centered modal jumps up and down under the user's eyes.
- Elevated `--ev-surface`, `1px solid var(--ev-border-strong)`, `13px` border radius, soft scrim backdrop, generous drop shadow, `overflow: hidden`.

### Prompt bar

- Single horizontal bar (`padding: 12px 14px`, `border-bottom: 1px solid var(--ev-border)`):
  - **Left:** monospace prompt pill (`M-x` or `SPC SPC`) in `--ev-accent` on `--ev-accent-soft` (`border-radius: 6px`, `padding: 2px 8px`).
  - **Center:** borderless input (`font-size: 15px`, `flex: 1`, `outline: none`), calm placeholder (`Search commands or initials (e.g. "mrt")…`), `autocomplete="off"`, `spellcheck="false"`.
  - **Right:** live match count (`12/84`) in `11px` muted `tabular-nums` monospace.

### Candidate list and row anatomy

- Scroll container (`max-height: min(50vh, 420px)`, `padding: 6px`), scrolling the active row smoothly with `scrollIntoView({ block: 'nearest' })`.
- **Row layout** (`min-height: 38px`, `padding: 7px 11px`, `border-radius: 8px`, horizontal flex with `10px` gap):
  - **Active row:** subtle `--ev-hover` fill + crisp `3px` left accent bar (`box-shadow: inset 3px 0 0 var(--ev-accent)`). High legibility, zero harsh inverted blocks.
  - **Primary slot (left, `flex: 1`, `min-width: 0`):**
    - Command label in primary ink (`13.5px`), with **matched query substrings/initials highlighted** in semibold `--ev-accent` (subtle mark tint).
    - Secondary inline or tucked muted metadata (`12px`): short description or group/category tag and muted monospace `:command-id` (truncated gracefully).
  - **Shortcut slot (right, `flex-shrink: 0`):**
    - Active keycap badge(s) (`SPC` `b` `p` or `p`), right-aligned so every `M-x` lookup teaches the user the keybinding.
    - If unavailable: muted pill stating the `need` (`needs open item`) and slightly dimmed row opacity (`0.65`).
- **Empty state:** centered calm message (`No matches for "xyz"`) with `padding: 24px 16px`.

### Ranking, orderless matching, and history

Implement Doom's `counsel-M-x` / `amx` + `orderless` ergonomics:

- **Empty query ranking (`amx`):**
  1. Most recently executed commands (top 7, most recent first).
  2. Remaining commands by descending execution count.
  3. Shorter labels first.
  4. Alphabetical tie-break.
  Persist recency, counts, and typed input history locally.
- **Multi-word filtering (`orderless`):**
  - Split query on whitespace; **every** token must match in any order.
  - Each token matches if it appears as a case-insensitive substring, matches as a safe case-insensitive regular expression (silently falling back to plain text if invalid regex syntax), or matches the label's word initialism (`mrt` matches `Mark read toggle`).
  - Search labels, IDs, aliases, details and bindings. Rank exact and prefix name matches before aliases, alternate keyboard-layout spellings and weaker matches. Break ties by existing recency/frequency and label length.
  - For bilingual names, try both layout directions and deduplicate by stable identity. Search notes and free text literally. Apply visibility rules before matching or displaying snippets. Clearing a list filter restores the advertised source order and the cursor by identity; ignore results from older queries.
- **Input history:** `M-p` steps backward through previously submitted queries; `M-n` steps forward back to the user's current unsent draft.

### Palette keyboard controls and focus restoration

- Move selection: `C-n`, `C-j`, `Down`, `Tab` (next, wrapping); `C-p`, `C-k`, `Up`, `S-Tab` (previous, wrapping).
- **First match is active:** the leading result is selected when the picker opens and after every query edit, so typing then `RET` takes it.
- Execute: `RET` (runs the active item and records it in history).
- Escape or `C-g` closes the palette at once and restores the invoking control and caret. A command palette stays in text entry and moves its selection with `C-n` / `C-p`, so it needs no Normal step. A filterable list, such as unread items or all bindings, follows the Escape ladder instead.
- **Subtle footer bar** (`padding: 7px 14px`, `border-top: 1px solid var(--ev-border)`, `font-size: 11px`, `--ev-muted`, `--ev-sunken` background): `C-n/p` move · `RET` run · `M-p/n` history · show the current Escape action.

## Surface 5: Supporting Visual Surfaces

### Avy / Link Hints (`f`, `y f`, `SPC j`)

- **High-contrast tag badges:** positioned at the top-left corner of each visible interactive target (clamped inside the viewport). Warm amber-gold fill (`#ffd84a`) with deep dark ink (`#1c1917`), `1px` crisp border (`#b89200`), bold `11px` uppercase monospace, `border-radius: 4px`, subtle shadow so tags pop cleanly over both light and dark UIs (≥ 7:1 contrast).
- **Typed-prefix narrowing:** highlight already-typed characters inside matching tags (for example crimson `#b91c1c` or dimmed opacity) and hide non-matching tags immediately.
- **Home-row prefix-free alphabet:** generate labels from home-row-first letters (`a s d f g h j k l e w c m p`), ensuring no label is a prefix of another.
- **Safe live targeting:** capture target identities at activation, filter out scroll-clipped or occluded elements, re-verify the live target at invocation, and cancel cleanly if the target moved or disappeared.

### Focused pane indicator and post-`Esc` selected input

- **Active pane:** mark the focused pane with a clean `2px` top (or left) accent border (`var(--ev-accent)`) so the user sees instantly which pane `j`/`k` controls. Avoid loud full-perimeter boxes.
- **Selected text box:** when `ESC` exits Insert mode, apply a quiet `1.5px dashed var(--ev-accent)` outline to that input so the user knows `i` jumps straight back into it. Remove the outline on the next `ESC` or focus move.

### Visible shortcuts and disabled explanations

- Put the effective key beside each button label, including Back and Cancel. Give icon buttons a visible keycap and an accessible name. A focus-only Return cue appears when the button owns focus.
- **Single-source disabled state:** derive `disabled` and its accessible explanation together; make the reason available to keyboard users, and scope hover highlights to `:hover:not(:disabled)`.

### User keymap validation banner

- When the user's `keys.edn` fails validation, keep built-in defaults active and display a calm, non-blocking warning strip at the top of the workspace naming the exact map, chord, line number, and problem (`keys.edn line 12 (:normal "SPC b"): unknown command :pin-all. Using built-in defaults.`).

## Anti-Patterns: Never Ship These

- **Toast-only mode feedback:** hiding the current mode or pending prefix behind a 1.5-second toast instead of a persistent modal status line.
- **Status line covering content:** fixed bottom bars without reserved bottom spacing, clipping the last item or composer.
- **Centered floating which-key modal:** covering what the user is reading instead of docking cleanly above the status line.
- **Raw command IDs in which-key or cheatsheet:** displaying `:open-archived-chats` instead of `Archived chats`.
- **One giant unsorted cheatsheet:** dumping 100+ raw bindings into `?` instead of separating a curated task-card cheatsheet (`?`) from the live binding reference (`SPC h b`).
- **Hardcoded key strings in UI:** tooltips, cheatsheets, or help text that drift the moment a binding changes or a user edits `keys.edn`.
- **Jumping `M-x` modal:** vertically centering `M-x` so the prompt jumps around while typing, or omitting right-aligned keycap hints and `amx`/`orderless` ranking.
- **Clashing or low-contrast chrome:** bolting a dark-only palette onto a light app, using neon inverted selection rows, or rendering cramped keycaps without monospace alignment.
- **Silent failure on unavailable or unknown keys:** doing nothing when a command's `when` fails, or letting an unknown prefix ending fall through to run a Normal-mode single key.
