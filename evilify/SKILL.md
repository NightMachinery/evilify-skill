---
name: evilify
description: Add Doom Emacs or Vim-style modal keyboard operation to an existing application, including leader sequences, command discovery, or Avy-style hints. Use for designing, implementing, or auditing a coherent keyboard interface across app actions.
---

# Evilify an existing application

Use the application's native action handlers and controls. First identify its panes, text inputs, modal layers, keyboard hooks, and actions users reach by mouse. Read the user's keymap references within their authorized scope. Preserve the exact keys they requested.

## Design the keyboard interface

- Give each command a stable name, a user-facing label, a brief description, an applicability check and an invocation. Keep actions with the feature that owns them. Bind key sequences to command names. Drive execution, which-key, the command palette, and help from the same commands and bindings.
- Choose letters by mnemonic first, then by reach. Keep a letter the user already knows: their editor's group letters (Doom's `SPC b`, `SPC o`, `SPC w`) and a command's own short key reused under the leader (`p` pins, so `SPC b p` pins). Among letters that are otherwise equally good, the home row wins, then the bottom row, then the top row, so a group used every day goes on `d` rather than `p`.
- State where keys become commands and where they enter text. Scope automatic interception to the focused surface and active mode. Let text inputs, composition, completion, modal dialogs and platform shortcuts keep their established behavior. Cancel unfinished sequences on context changes. Avoid shortcuts that send, mark read, delete or otherwise commit while a user is just moving a cursor.
- Decide how a prefix waits, how `Esc` backs out, how unknown endings are handled and whether repeat is admitted. Check conflicts between complete chords and prefixes. Model navigation and actual activation separately when activation has effects such as opening or marking an item read.
- Give help a clear route. For Doom-like defaults, use `SPC ?` for help, `SPC SPC` for command search, `SPC h k` for reading a chord, and `SPC h f` for describing a command. Describe-key captures without invoking the key. Show each command's applicable bindings and explain unavailable actions when the user is looking for them.
- Distinguish normal command search from hinted targeting. Hints label real targets in the active visible region, with a specific intent such as open, select or copy. Capture stable identities, resolve the same live target at invocation, and cancel if it was replaced, hidden, or moved outside scope. Apply the app's normal protections for copying and actions. Prefer supported action and focus APIs to global coordinate clicks.
- Write into the app's developer docs that every new page, panel or action gets its command in the same change, since gaps appear in features added after the keyboard pass. Link those docs from the project's agent instructions (`AGENTS.md`) so agents adding features find the rule.
- Let a local keymap remap command IDs when the task calls for customization. Validate it before applying it. Adding a domain action belongs in code or the app's existing trusted extension system. Keep configuration parsing separate from command execution.

## Verify keyboard access

Use the app's normal paths for the keyboard tests. Exercise focus transitions, cancel, repeated keys, incomplete sequences, IME composition, account or window switches, a modal layer, stale hints, and default application behavior with the mode off. Check that entering a command through a chord and through the palette calls the same handler, with the same protections and confirmations. Check hint reachability with scroll clipping and overlay occlusion.

For broad coverage, inventory visible workflows and give each an accessible route through a command, native focus and activation, or an explicit hint target. Verify representative workflows end to end and keep gaps visible. Keep screenshots and live-account fixtures out of shared repositories.
