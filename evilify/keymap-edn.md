# A shared EDN keymap format

Use this format when an evilified app lets the user remap its keys. Purple
(Telegram Desktop) reads it, and agent-desk is moving its keys to it. Apps that
follow it can share documentation, tooling and habits.

Terms used below:

- **Keymap file:** an EDN file in this format. The app ships its whole default
  keymap as one; the user's keymap file holds only changes and overlays it.
- **Named map:** one binding table in a keymap file, such as `:normal`. The app
  defines which names exist.
- **Chord:** a key sequence written as a string, such as `"SPC o a"`. Each
  space-separated part is a **stroke**.
- **Effective map:** the bindings the app dispatches from at a given moment,
  after overlay and context precedence. Dispatch, which-key and help all use it.
- **Portable subset:** the part of this format every conforming app accepts.
  Anything beyond it is an app-specific extension the app must document.

## The file

```clojure
; Comments run from ; to the end of the line. #_ discards the next form.
{:normal {"SPC o a" :open-archived-chats   ; chord -> command: bind it
          "g s s"   nil}                   ; chord -> nil: remove this entry
 :insert {"C-[" :enter-normal-mode}}
```

- The file holds exactly one top-level form, a map from named-map keywords to
  binding maps. A second form after it, or none, is an error.
- A binding map maps a chord to either:
  - an unqualified command keyword (`:open-archived-chats`, not
    `:purple/open-archived-chats`): bind it;
  - `nil`: remove that exact entry (see Overlay).
- Read it with a non-evaluating reader: no tagged literals, no reader
  evaluation. Prefer a maintained EDN library (edn-data in JavaScript,
  `clojure.edn` on the JVM) over a hand-written reader; escapes, discards,
  duplicate keys and trailing forms all need real tests.
- Reject a key that appears twice in one binding map, whether the two strings
  are identical (raw duplicate) or only equal after normalizing (`"C-x"` and
  `"Ctrl+x"`, a normalized duplicate).

## Chords

The portable subset, which every example in shared docs sticks to:

- Strokes separated by single spaces: `"SPC o a"`, `"g s s"`, `"C-w h"`.
- Modifiers `C-` (Control), `M-` (Alt/Option), `S-` (Shift), `s-`
  (Super/Command).
- `SPC` and `RET` for Space and Return.
- Letters in lowercase, and a shifted letter as `S-` plus the lowercase
  letter: `"S-g"`.
- A shifted punctuation key as the symbol it types: `"?"`, `"C-+"`.
- At most four strokes.

Named keys beyond that are extensions. Use the Doom spelling as the recommended
name in new implementations, and document which spellings and aliases the app
accepts. Known spellings:

- Backspace: Doom `DEL`, alias `Backspace`. Emacs `DEL` means Backspace, not
  forward delete.
- Forward Delete: Doom `<delete>`, alias `Delete`. Keep it distinct from
  Backspace.
- Escape and Tab: Doom `ESC` and `TAB`. Many apps reserve them (see Reserved
  keys), so check before binding them.
- Arrows, Home and End: Doom `<up>` `<down>` `<left>` `<right>` `<home>` `<end>`;
  Purple `Up`, `Down`, `Left`, `Right`, `Home`, `End`.
- Page keys: Doom `<prior>` `<next>`; Purple `PgUp` `PgDown` (not `PageUp`).
- Function keys: Doom `<f1>` to `<f35>`; Purple `F1` to `F35`.
- Long modifier forms: `Ctrl+`, `Alt+`, `Shift+`, `Super+`.

Normalize chords before comparing them, so that every accepted spelling of a
chord compares equal. Named keys are case-insensitive. An uppercase letter is
not portable: Purple reads `G` as `g`, while an app may document it as an alias
for `S-g`.

## Overlay

Overlay happens within each named map, before context precedence:

- A user entry replaces the default entry with the same normalized chord in the
  same named map.
- `nil` removes only that exact entry in that map. It does not remove the
  chords that extend it (`"g s" nil` leaves `"g s s"` alone), and it does not
  hide a binding of the same chord in another named map.
- Within one named map, a command chord that is also a prefix of another chord
  (`"g"` and `"g s"`) is an error, whichever entry comes first.

## Context precedence

When several named maps are active together (a pane map and `:normal`, say),
the app documents their order, most specific first. Then:

- Freeze the active maps at the first stroke of a sequence, so a focus change
  halfway through a chord cannot change what it means.
- Build the effective map from the highest-priority map down. A higher binding
  shadows a lower one with the same chord, and also a lower one it collides with
  as a prefix: a higher `"g"` command hides a lower `"g s s"`, and a higher
  `"g s x"` hides a lower `"g s"` command.
- Which-key and help show the same effective map that dispatch uses.

## Insert mode

- The first stroke of an `:insert` chord needs Ctrl, Alt or Super. Shift alone
  does not count, since it types text. Later strokes may be plain keys.
- Native shortcuts, text input and IME composition keep working.
- A focus or mode change cancels a pending prefix.

## Reserved keys and required bindings

Each app documents the chords a user file may not take or remove, and the
bindings it must keep. Purple, for example, protects four help and palette
chords, reserves `C-g` and a bare leading `?`, passes Tab and Backtab through,
and requires at least one `:insert` binding that enters Normal mode. A user
file that breaks these rules is invalid.

## Validation

Reject the whole user file, and keep the defaults, when it has any of:

- not exactly one top-level map;
- an unknown named map or an unknown command;
- a chord that does not parse, or a raw or normalized duplicate;
- a command/prefix conflict within one named map;
- a reserved chord taken, or a required binding removed;
- a value the app does not support (a string, unless it supports the label
  extension below).

Recommended, not required: name the offending entry and its location in the
error, and show it where the user will see it, not only in a log. (Purple
falls back atomically but logs a generic message.) Size limits are the app's
choice (Purple: 16 KiB and 128 entries).

## Optional extensions

Apps that only support the portable subset reject these keys and value types,
so omit them from keymap files meant to stay portable across every app.

### Prefix labels

An app may let a string value label a prefix in which-key and the status line:

```clojure
{:normal {"SPC o" "open"}}
```

- A label is presentation only. It names a prefix that has effective children;
  it is never a command and never creates a prefix. A label on a command chord,
  or on a chord with no effective children, is an error.
- Write the name alone. A `+` marker in front of prefix names is the app's
  decoration, not part of the label.

### Localleader chords (`:localleader`)

An app with object- or page-specific context maps (such as `:task`, `:media`,
or `:settings`) may declare the chords that activate them in one top-level
vector:

```clojure
{:localleader ["," "SPC m"]
 :task {"d" :task-done
        "D" :task-drop}}
```

- Context maps store only the suffix strokes typed after the localleader, so a
  user can move the localleader in one edit without rewriting every context
  binding.
- Every entry in `:localleader` must parse as a valid non-empty chord, must
  not collide with a reserved chord, and is validated as a prefix across active
  context maps.

### Deliberately keyless commands (`:keyless`)

An app with keyboard coverage tests may list commands intentionally left
without a binding under a top-level `:keyless` vector of unqualified command
keywords:

```clojure
{:keyless [:bulk-archive-all :cycle-theme-dark]}
```

- Every entry must name a known command.
- Coverage tests accept commands in `:keyless` without requiring a chord, and
  `SPC h f` (describe command) reports them as "no key (on purpose)" while
  keeping them runnable via `M-x`.

## Starter user file

Recommended, not required: when the app starts and the user's keymap file does
not exist yet, create it from a concise starter template (see
[templates/keys-template.edn](templates/keys-template.edn)) containing short
comments on the syntax, the app's named maps, how to look up command IDs
(`SPC h f`), and two or three commented-out examples. Never overwrite an
existing user file, and name the file path at the bottom of the `?` cheatsheet.

## What each app documents

- where the user's keymap file lives (and whether it is auto-created when
  missing);
- its named maps, which are active together, and their precedence;
- the chord spellings and aliases it accepts beyond the portable subset;
- its reserved chords and required bindings;
- its size limits;
- which optional extensions it supports (prefix labels, `:localleader`,
  `:keyless`).
