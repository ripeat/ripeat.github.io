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

Check it worked: `ripeat version` prints the version, and `ripeat` with no arguments prints the command list.

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
