# seat

`seat` is a tiny command-line utility for switching accounts in Claude Desktop on macOS.
Keep your personal and work accounts signed in and switch between them with one command,
without signing out and back in.

## Small enough to read before you run it

A tool that handles your Claude sign-in shouldn't need blind trust, so `seat` was built
to be as small as possible. The whole tool is [one zsh script](seat): about 250 lines of
code, with comments that explain every step. You can actually read all of it before you
run it. Start with the comment at the top: it lists everything the script touches.

- **It only moves folders.** A switch renames Claude's data folder while Claude is
  closed. `seat` never opens your cookies, tokens, or chats, never touches the Keychain,
  and doesn't modify Claude.app.
- **No network, no `sudo`, no daemons.** It doesn't connect to anything, doesn't need
  admin rights, and no part of it keeps running after a switch.
- **No dependencies.** It uses only zsh and tools that come with macOS. Nothing is
  compiled, so the code you read is exactly the code that runs.
- **It never deletes your data.** A switch is two renames, and `seat rm` moves a profile
  to the Trash.

## Install

```
brew install andwhy/tap/seat
```

Homebrew installs the script as is, so you can read exactly what you've installed before
the first run: `less "$(command -v seat)"`. Update with `brew upgrade seat`. Without
Homebrew, clone the repository and symlink `seat` into any folder on your `PATH`.

## Usage

```
seat                        list profiles
seat add <name>             create an empty profile
seat use <name>             switch: Claude quits and reopens with this profile
seat rename <old> <new>     rename a profile
seat rm <name> [-y]         delete a profile (moves it to the Trash)
seat current                print the active profile's name
```

First-time setup:

```
seat rename default personal   # your current Claude data is the "default" profile
seat add work
seat use work                  # sign in to your work account once
seat use personal              # switch back, no sign-in needed
```

## How it works

Claude Desktop keeps everything tied to an account (cookies, tokens, Code and Cowork
chats, MCP settings) in `~/Library/Application Support/Claude`. The release build
can't be pointed at another folder, so `seat` swaps folders while Claude is closed:

- the active profile sits at the standard path;
- the others wait in `~/Library/Application Support/Claude Profiles/<name>`;
- each profile contains a `.seat-profile` marker with its name.

A rename on the same volume is instant; no data is copied.
Claude is closed with SIGTERM, which Electron handles the same way as Cmd+Q.

`~/.claude` (Claude Code settings, skills, and history) is shared by all profiles.

## Caveats

- Switching quits Claude, which stops any active Code and Cowork sessions.
- If you run `seat use` from inside Claude (for example, from the Code tab), the switch
  happens in the background. Log: `Claude Profiles/.seat.log`.
- Each profile has its own Cowork virtual machine (`vm_bundles` in the profile folder):
  the first time you start Cowork in a profile, Claude downloads about 1.3 GB and unpacks
  it to about 12 GB on disk, and it updates each profile's copy separately. The VM isn't
  tied to an account; it's per-profile only because seat swaps Claude's whole data folder.
  A profile removed with `seat rm` keeps taking up that space until you empty the Trash.
- Don't launch a second Claude instance (`open -n`): it would use the same profile folder.

## Uninstall

To undo everything: run `seat use default` (or whatever name you gave the original
profile), delete `~/Library/Application Support/Claude Profiles`, then run
`brew uninstall seat`.
