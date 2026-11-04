# Getting started

## Install

**macOS and Linux** (Homebrew):

```bash
brew install ripeat/tap/ripeat
```

**Windows** (Scoop; it needs `git` to add a bucket):

```powershell
scoop install git
scoop bucket add ripeat https://github.com/ripeat/scoop-bucket
scoop install ripeat
```

Check it worked: `ripeat version` prints the version, and `ripeat` with no arguments, in a terminal, asks what you want to do (below); in a script or a pipe it prints the command list.

**Update:** `brew update && brew upgrade ripeat/tap/ripeat`, or `scoop update ripeat`. `ripeat version --check` tells you whether a newer release is out.

**By hand:** each release at [github.com/ripeat/releases](https://github.com/ripeat/releases/releases) has one archive per system (`darwin`, `linux`, `windows`, on `amd64` and `arm64`) and a
`checksums.txt`. Check the download against it before you run it. Programs downloaded in a browser are not signed (ripeat does not do code signing), so Windows SmartScreen or macOS Gatekeeper may warn about them;
installing with Homebrew or Scoop avoids that.

## Where ripeat keeps its files

The folder is `$RIPEAT_HOME`, or `ripeat` inside your user config folder
(`~/.config/ripeat` on Linux, `~/Library/Application Support/ripeat` on macOS, `%AppData%\ripeat` on Windows).

| Path | What it is |
|---|---|
| `state.json` | What ripeat installed and the journal of runs (the last 20) |
| `state.json.lock` | Held while a command that writes state is running |
| `recipes/*.yaml` | Your own recipes; they win over built-in ones and over those of recipe packs |
| `profiles/*.yaml` | Saved profiles, run by name with `--profile` ([Profiles: your list](./profiles)) |
| `drafts/*.yaml` | Recipes an AI assistant prepared, waiting for your approval; never read as recipes ([Drafts from an assistant](./drafts)) |
| `taps/<name>/`, `taps.json` | The recipe packs you added: their recipes, and each pack's repository, pinned commit and settings ([Recipe packs (taps)](./taps)) |
| `installers/<name>/` | The `.msi` file kept so it can be uninstalled later ([Vendor installers](./installers)) |
| `apps/<name>/` | Programs installed from downloads ([Downloads (GitHub, URL)](./downloads)) |
| `bin/` | Links to those programs. Add this folder to your `PATH` yourself; ripeat never edits your shell files |

`ripeat.yaml`, `ripeat.lock` and `ripeat.policy.yaml` live in the folder you run ripeat from.

## Start here: the menu

Type `ripeat` in a terminal and it asks one question:

```
What would you like to do?
  1. Create a profile: a list of tools, written in your editor
  2. Use a saved profile (none yet)
  3. Install tools by name
  4. See all commands
```

- **1** asks for a name and opens your editor (`$VISUAL`, then `$EDITOR`) on a blank profile that starts with a link to [how to write one](./profiles). Leave the editor to save it; it is checked first.
- **2** lists your saved profiles and, for the one you pick, shows its plan and risk and asks before installing anything.
- **3** takes tool names and does the same: the plan and its risk, then the question. A name with no recipe offers to look it up.
- **4** prints every command. An empty answer, `q`, or the end of the input leaves without doing anything.

The menu only runs the ordinary commands (`profile create`, `install`), so every question, risk rating and confirmation they have still applies, and there is no way to skip them from here.

## First run

Look first, then act. `plan` is `install --dry-run`: it prints the plan and changes nothing.

```bash
ripeat recipes              # what ripeat knows how to install
ripeat plan git node        # what would happen
ripeat install git node     # do it
ripeat list                 # what ripeat manages
ripeat rollback             # undo the last run
```

With no tool names, `install` reads `./ripeat.yaml` if it exists ([Profiles: your list](./profiles)).

## Every command

The [command reference](./commands) lists every command, what it changes, and its options.

`--dry-run` is available on `install`, `uninstall`, `rollback` and `upgrade`, and a
dry run never writes state. `install`, `uninstall` and `upgrade` show what they will do and **ask before
changing anything**; add `--yes` to skip the question (required when there is no terminal).
