# seat

Switch Claude Desktop accounts on macOS without signing in again.

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
- Each profile has its own Cowork virtual machine: the first time you start Cowork
  in a new profile, it's downloaded again (~10 GB of disk space).
- Don't launch a second Claude instance (`open -n`): it would use the same profile folder.

## Install

```
brew install andwhy/tap/seat
```

Update with `brew upgrade seat`. Without Homebrew, clone the repository and symlink
`seat` into any folder on your `PATH`.

## Uninstall

To undo everything: run `seat use default` (or whatever name you gave the original
profile), delete `~/Library/Application Support/Claude Profiles`, then run
`brew uninstall seat`.
