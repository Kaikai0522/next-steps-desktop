# next-steps-desktop

A fork of [next-steps](https://github.com/anthropics/claude-plugins-community/tree/main/next-steps) by **Thariq Shihipar** (MIT). It adds one thing: the `1` / `2` / `3` / `0` number keys also work in the **Claude desktop app**, not only in the terminal.

After each turn, it suggests up to three next prompts above the input box.

```
next:
  1: run the tests you just wrote
  2: do the same for the settings page
  3: /code-review high
  0: dismiss
```

Press `1`, `2` or `3` from an empty prompt box (or click one) and that prompt is written into the box as a draft. Edit it, then press Enter yourself. `0` dismisses. The top suggestion also shows as the box's dim ghost text, so Tab takes it.

The plugin never submits a prompt on its own.

## Install

At the prompt of a Claude Code terminal session:

```
/plugin install next-steps-desktop --marketplace Kaikai0522/next-steps-desktop
```

Answer `y` to add the marketplace, then pick a scope (user is the usual one).

From a shell:

```
claude plugin install next-steps-desktop --marketplace Kaikai0522/next-steps-desktop
```

In the desktop app, where `/plugin` does not run: click **+** next to the prompt box, then **Plugins**, **Add marketplace** with `Kaikai0522/next-steps-desktop`, then **Add plugin**.

If you already have the original `next-steps` installed, uninstall it first so you don't get two suggestion bands:

```
claude plugin uninstall next-steps@claude-community
```

## What this fork changes

The original binds the number keys to the band's buttons (`hotkey`). In the terminal, a bare digit typed into an empty prompt box reaches those buttons. In the desktop app the prompt box is the app's own, so the digit never reaches them and only clicking works.

This fork adds two hooks in `hooks/register.tsx`:

- `prompt.edit`: while suggestions are shown and the prompt box is empty, typing `1`/`2`/`3` replaces the box with that suggestion instead of the digit, and `0` dismisses.
- `prompt.change`: a fallback for surfaces that sync the whole box rather than send edits: when the box holds just that digit, it is replaced with the suggestion (or cleared on `0`).

With no suggestions shown, or text already in the box, digits type as usual. Everything else is the original plugin, unchanged.

## How it works

It is a function-hooks plugin (`hooks/register.tsx`):

- `turn.complete`: forks the session with `$.model.fork` to ask for likely next prompts. The fork shares the session's prompt cache, so it costs about one short reply.
- `$.command.list`: the session's skills and slash commands (plugin, user and MCP ones with their descriptions) go into the fork's question, so a suggestion can be `/skill arguments`. A suggestion that names a command the session does not have is dropped.
- `ui.render` on `AbovePrompt`: draws the suggestions as buttons.
- A press calls `$.prompt.fill`; the top suggestion goes to `$.prompt.suggest`.
- `prompt.edit` / `prompt.change`: the number keys in the desktop app (this fork).
- `turn.start`: hides the suggestions.

## Options

| Option | Default | What it does |
| --- | --- | --- |
| `minAnswerChars` | `80` | Skip suggestions after answers shorter than this |
| `suggestSkills` | `true` | Tell the suggester which skills and slash commands the session has |

## License

MIT. Original plugin © Thariq Shihipar; desktop number-key changes © Kaikai0522. See [LICENSE](LICENSE).
