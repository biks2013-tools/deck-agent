# deck-agent

A local agent for presentations built with the `nbg-design` skill. You open a deck through it, a panel appears in
the page, and what you type there goes to an agent that carries the skill and edits the deck file on disk: change
slides, add, move, copy or extract them, pick up edits you made in the page, read Word, Excel and PowerPoint
files, search your folders and the web, and export to PowerPoint or PDF.

This repository holds the releases. Each one is a prebuilt package: nothing is compiled on your machine.

## Install

```sh
# Homebrew (brings its own Node.js)
brew install biks2013-tools/deck-agent/deck-agent

# without Homebrew, on macOS or Linux (needs Node.js 22.19.0 or newer, curl and tar)
curl -fsSL https://github.com/biks2013-tools/deck-agent/releases/latest/download/install.sh | bash
```

The second form installs to `~/.local/share/deck-agent` and puts the command at `~/.local/bin/deck-agent`.
There is no installer for Windows.

## Set up and run

```sh
deck-agent init      # writes ~/.tool-agents/deck-agent/.env with a generated access token
                     # fill in the other settings: every one is required
deck-agent check     # names every setting that is missing or invalid
deck-agent serve     # keeps running; in the background: brew services start deck-agent
deck-agent open /path/to/deck.html     # in another terminal: opens the deck with the agent panel
```

## Attach a presentation

```sh
deck-agent admin     # opens the admin screen (the host must be running)
```

The admin screen lists the presentations in your deck folders. **Attach** writes a small connector into the HTML
file. From then on the presentation, opened straight from the disk, shows an **Agent** button that brings it to
the agent. **Detach** removes the connector and leaves the file exactly as it was.

The connector works for that file, at that path, on your computer only. It does not contain your access token,
so a presentation you send to someone else gives them no access. A file you move or copy needs attaching again.

You also need:

- the `nbg-design` skill on the machine (`DECK_AGENT_SKILL_DIR` points to the folder that holds its `SKILL.md`);
  it is available from [BikS2013-coding-agents/nbg-design](https://github.com/BikS2013-coding-agents/nbg-design);
- Google Chrome, for the skill's checks and exports;
- an API key for a model provider and a [Tavily](https://tavily.com) key for web search.

As a background service the host does not see your shell's variables, so the provider's API key goes in the
`.env` file as well. The full reference is installed with the package, in `docs/tools/deck-agent.md`.

## What to know before you use it

- The host listens on `127.0.0.1` only and every request needs the access token.
- The agent has a shell and can run commands as you, in any folder. The folders you configure limit its deck,
  Office and search tools, not the shell. Do not point it at documents or web pages you do not trust.
- It keeps a copy of a deck before every change; the panel's Undo restores it.
- It is built on [pi-durable](https://www.npmjs.com/package/@earendil-works/pi-durable), which is experimental.
