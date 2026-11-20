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
```

Then run it in one of two ways.

One step, with a desktop window:

```sh
deck-agent app [folder]     # starts the host and opens the admin window
```

Or the host and the admin screen separately, in the browser:

```sh
deck-agent serve     # keeps running; in the background: brew services start deck-agent
deck-agent admin [folder]              # in another terminal: the admin screen as a web page
deck-agent open /path/to/deck.html     # or open one deck directly, with the agent panel
```

## The desktop window

`deck-agent app` starts the host and opens a window that looks like a native macOS application. Choose a folder
with **Choose Folder…** (or File > Choose Folder…, Cmd+O); the window lists the presentations of that folder and
of the folders inside it, and remembers the folders you used. For each presentation there is **Attach** /
**Detach**, **Open** (in the default browser, with the agent panel; a double click does the same) and Show in
Finder. The window also has a search field, a Settings view and the installed version.

- Quitting the application (Cmd+Q) stops the host. Closing the window leaves both running; the Dock icon opens
  the window again. Ctrl+C in the terminal stops both.
- When a host with the same settings is already running (for example as a background service), the command uses
  it and leaves it running. It must be the same version; otherwise the command says so and stops.
- The window is built with Electron. The first `deck-agent app` downloads Electron (about 110 MB) from Electron's
  releases, once per installed version, so that first start needs the network. On macOS the application is then
  named "Deck Agent" and signed for your computer.
- Tested on macOS (Apple Silicon). Linux and Windows are untested.

## The macOS application

```sh
deck-agent install-app                  # builds /Applications/Deck Agent.app
deck-agent install-app ~/Applications   # or in another folder
```

"Deck Agent" is then a regular application: open it from the Launchpad, Spotlight or the Finder, and keep it in
the Dock (right-click its Dock icon, Options > Keep in Dock). It starts the host by itself and shows the same
window as `deck-agent app`; quitting it stops the host.

- It has no shell behind it: it reads `~/.tool-agents/deck-agent/.env` only. A key that is only exported in your
  shell profile, such as the model provider's API key, must be in that file. `install-app` names the settings
  that are missing there; in the window's Settings, "Keep in the settings file" writes them.
- When it cannot start, a dialog says why. The host's log is `~/Library/Logs/Deck Agent/host.log`.
- It follows the installed deck-agent: after `brew upgrade deck-agent` it runs the new version. Run
  `deck-agent install-app` again to renew the Electron inside it, or after an upgrade made with `install.sh`.
- The application is built for your Mac (about 250 MB, signed for that computer); it is not a file to pass on.

**Settings** in the window is a form over every setting: the provider and model from a list, folders with the
folder chooser, secrets typed blind (they are never shown; leave one empty to keep it). **Save** checks the whole
configuration first and writes nothing if anything is wrong. A valid change goes to the .env file the value
comes from (`~/.tool-agents/deck-agent/.env`, or a local `.env` that already defines it), changing only its own
line, and the host restarts so that it applies; the window comes back on Settings. A value fixed by a
command-line flag cannot be changed there. The web admin and the agent panel show the same settings read-only.

## Attach a presentation

```sh
deck-agent admin ~/Documents/decks     # opens the admin screen on that folder (the host must be running)
deck-agent admin                       # the same, on the folder you are in
```

The admin screen lists the presentations in that folder and the folders inside it; another folder can be typed
in the screen. A presentation may be in any folder. **Attach** writes a small connector into the HTML
file. From then on the presentation, opened straight from the disk, shows an **Agent** button that brings it to
the agent. **Detach** removes the connector and leaves the file exactly as it was.

The connector works for that file, at that path, on your computer only. It does not contain your access token,
so a presentation you send to someone else gives them no access. A file you move or copy needs attaching again.

The admin screen shows the installed version and whether a newer one is published, has a System / Light / Dark
theme switch and a Settings tab. The agent panel in a presentation can be resized by dragging its left edge, top
edge or top-left corner, and has a Settings tab of its own.

The panel's Activity tab shows what the agent did for each instruction, step by step: every model response with
its time, duration and tokens, the model's thinking, and every tool call with its arguments, its result, how long
it took and whether it failed. A request that is running is shown live. It lists the latest 30 requests.

You also need:

- the `nbg-design` skill on the machine (`DECK_AGENT_SKILL_DIR` points to the folder that holds its `SKILL.md`);
  it is available from [BikS2013-coding-agents/nbg-design](https://github.com/BikS2013-coding-agents/nbg-design);
- Google Chrome, for the skill's checks and exports;
- an API key for a model provider and a [Tavily](https://tavily.com) key for web search.

As a background service the host does not see your shell's variables, so the provider's API key goes in the
`.env` file as well. The full reference is installed with the package, in `docs/tools/deck-agent.md`.

## What to know before you use it

Upgrading from 0.4.0 or earlier: remove `DECK_AGENT_DECK_ROOTS` from your settings. It no longer exists, and
deck-agent refuses to start while it is set. Anyone signed in to the host on your computer can open and attach
any HTML file on it; the agent's deck and Office tools write only in the folder of the presentation they work on.

- The host listens on `127.0.0.1` only and every request needs the access token.
- The agent has a shell and can run commands as you, in any folder. The folders you configure limit its deck,
  Office and search tools, not the shell. Do not point it at documents or web pages you do not trust.
- It keeps a copy of a deck before every change; the panel's Undo restores it.
- It is built on [pi-durable](https://www.npmjs.com/package/@earendil-works/pi-durable), which is experimental.
