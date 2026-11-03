# Recipes

A recipe says how one tool is installed on each OS. The user's list says *what*
([Profiles: your list](./profiles)); recipes say *how*. Adding an OS means adding a backend
and `darwin:` sections; the core does not change.

Built-in recipes: `git`, `node`, `typescript`, `chrome`, `firefox`, `vscode`,
`docker`, `ripgrep`. `ripeat recipes` lists everything available, including your own.

## Where recipes come from

Three layers, lowest first, and a higher layer wins when two have the same name: the **built-in** recipes, the recipes of the **recipe packs** you
added ([Recipe packs (taps)](./taps); a pack cannot take a built-in name), and **your own** (below). `ripeat recipes check` shows each recipe's origin
(`shipped`, `added`, or `tap:<name>`).

## Your own recipes

To get a starting recipe for a tool, use `ripeat add` ([Install by name](./find-and-add)).

Put `*.yaml` files in `$RIPEAT_HOME/recipes/`. A recipe with the same `name` as a
built-in one replaces it.

## Format

```yaml
name: typescript
description: TypeScript compiler (global npm package)
requires: [node]            # installed first; "node@20" pins it
check: tsc --version        # succeeds if the tool is already on the machine
version: tsc --version      # optional; prints the installed version, defaults to check
platforms:
  linux:
    - { method: apt,    package: git }       # a list: first usable option wins
    - { method: dnf,    package: git }
  windows: { method: winget, package: Git.Git }   # or a single mapping
```

Platform keys are `linux`, `windows`, `darwin` (macOS). A tool without a section for the
current OS cannot be installed there. Unknown keys anywhere in a recipe are errors,
and `ripeat validate` ([Checking your files](./validate)) checks the rest before you install.

## Is a recipe still good? `ripeat recipes check`

A recipe says "install package X"; it does not pin a version, so "latest" never goes stale. What goes stale is the *how*:
a package renamed or removed, a project that renames its release files, a download address that no longer exists.

```bash
ripeat recipes check            # every recipe, for this machine's OS
ripeat recipes check git node   # only these
```

It asks the real sources and changes nothing (no install, no download of programs):

| Method | What is asked |
|---|---|
| `apt`, `dnf`, `pacman`, `brew`, `brew_cask`, `choco`, `winget`, `scoop`, `snap`, `flatpak` | the package manager, for each package named in the option (`winget` asks the Microsoft Store when the option has `source: msstore`) |
| `github_release` | GitHub, resolving the latest release as an install would: exactly one file matches `asset`, and a SHA-256 is published |
| `url`, `deb_url` | the server, for the file's headers only |
| `shell` | **not checked**: a shell command cannot be tried without running it |

Each option gets a line: `✓` works, `✗` broken (with the reason), `-` not checked (with the reason; for example a package manager
that is not on this machine). Each recipe gets one mark: `✓` every option works, `!` one is broken but another still works,
`✗` none works. The command exits with an error when a recipe is `✗`, so it can run in a script.

What to do: for a recipe you added, `ripeat add <name> --refresh` drafts it again and shows the change
([Install by name](./find-and-add)); for a built-in one, upgrade ripeat (`ripeat version --check`, [Versions and upgrades](./upgrades)), because
built-in recipes arrive with each release. ripeat never changes a recipe by itself and never pulls recipes from a feed: a recipe can
hold commands, so each one is something you or the ripeat project reviewed.

Only options for this machine's OS are checked, and only what the sources say today: a green line means the package or file
is there, not that installing it will succeed.

### Keys inside a platform entry

| Key | Used by | Meaning |
|---|---|---|
| `method` | all | How to install; see the table below |
| `package` | package managers; downloads | Package name or id. Several names separated by spaces install together (`nodejs npm`). For downloads it is the app's folder name |
| `scope` | all | `user` (needs no admin rights) or `system` (the default). See [Admin rights](./privileges) |
| `source` | `winget` | The catalog: `winget` (the default, the community one) or `msstore` (the Microsoft Store). The `package` is then the Store id (`9NT1R1C2HH7J`) |
| `url` | `deb_url`, `url` | Download address. `{os}`, `{arch}`, `{arch_gnu}` are replaced |
| `install`, `uninstall` | `shell` | Commands run through the platform shell |
| `repo`, `asset`, `checksum_asset`, `bins` | `github_release` | See [Downloads (GitHub, URL)](./downloads) |
| `kind`, `app`, `signer`, `unsigned`, `product` (and `url`, `sha256`) | `installer` | See [Vendor installers](./installers) |
| `sha256`, `bins` | `url` | See [Downloads (GitHub, URL)](./downloads) |
| `note` | `manual` | How to get the tool, shown when it is missing. Required |

## Install methods

| Method | Runs | Pin a version | Needs admin |
|---|---|---|---|
| `apt` | `apt-get install` | yes, `pkg=version` | yes |
| `dnf` | `dnf install` | yes, `pkg-version` | yes |
| `pacman` | `pacman -S`; removal uses `-Rs` so helper packages go too | no, warns | yes |
| `snap` | `snap install` | no, warns | yes |
| `brew` | `brew install --formula`; the current version only (an older one needs its own formula, such as `node@20`, in `package`) | no, warns | no: Homebrew refuses to run as root |
| `brew_cask` | `brew install --cask` (apps) | no, warns | no; an app's own installer may ask for your password |
| `flatpak` | `flatpak install` from Flathub; `--user` when `scope: user` | no, warns | unless `scope: user` |
| `winget` | `winget install`; `--scope user` when `scope: user`; `--source msstore` when `source: msstore` | yes | the installer asks (UAC) |
| `scoop` | `scoop install` | yes, `pkg@version` | no |
| `choco` | `choco install` | yes | needs an elevated terminal |
| `deb_url` | downloads a `.deb`, installs it with `apt-get` | no, warns; **not checksum-verified** | yes |
| `github_release` | downloads a release asset | yes, release tag | no |
| `url` | downloads a fixed URL | no, warns | no |
| `installer` | downloads a vendor `.dmg` (copies one app), `.pkg` (read first, refused if unsafe, undone by its receipt), `.msi` (runs `msiexec`) or `.exe` (runs it silently; undone through its entry in Add or remove programs), after checking its SHA-256 and signature; see [Vendor installers](./installers) | no, warns | dmg: no with `scope: user`; msi: yes |
| `manual` | nothing: a tool **you** install by hand. ripeat only runs the recipe's `check`, and shows the `note` when the tool is missing; see below | no | no |
| `shell` | your `install` command | up to the command | per `scope` |

A `shell` recipe can only be rolled back or uninstalled if it has an `uninstall` command.
Shell commands run through `sh -c` on Linux and `cmd /C` on Windows, exactly as written
(quotes included).

### `manual`: a tool you install by hand

For an app ripeat cannot or should not install (one that needs a licence, a vendor installer ripeat does not run, something your administrator hands out), a recipe can say so:

```yaml
name: excel
description: Microsoft Excel
check: test -d "/Applications/Microsoft Excel.app"     # required: succeeds when it is installed
platforms:
  darwin: { method: manual, note: "Install Microsoft 365 from https://www.microsoft.com/microsoft-365 (it needs a licence)." }
```

`ripeat add excel --manual --check '...' --note '...'` writes it for you ([Install by name](./find-and-add)). How it behaves:

- **If `check` succeeds,** the tool is on the machine: it is recorded as already installed and left alone, like any tool that was there first.
- **If not,** the plan shows `for you to install: <note>`, `assess` lists it as "by hand" with no risk level, and `install` carries on with everything else, then ends with
  **"Left for you to install by hand"** and the notes. Install it yourself, run ripeat again, and it sees the tool.
- **A tool that `requires` a missing manual tool is refused up front** ("addin needs excel, which ripeat cannot install: install it yourself first"), and nothing is changed.
- **ripeat never installs or removes it,** a manual step needs no administrator rights, and the administrator policy's method rules do not apply to it, because it runs nothing.
- **The `check` is required** (a manual recipe without one could never tell the tool is there). It is a command like any `check` ([What `check` does](#what-check-does)).
- An assistant connected through `ripeat mcp` cannot draft a manual recipe: its `check` is a command, and drafts are limited to what ripeat can write itself ([Drafts from an assistant](./drafts)).
  `ripeat_resolve` reports such a tool as `install_by_hand`.

## Fallbacks: first usable method wins

When a platform lists several options, ripeat uses the first whose tool is on the
machine (`apt` needs `apt-get`, `dnf` needs `dnf`, and so on; `shell`, `url` and
`github_release` always count). If none is available the plan stops with an error
naming the methods it needed, before anything changes. `--scope` narrows the
options further ([Admin rights](./privileges)).

## Dependencies: `requires`

Missing dependencies are added to the plan automatically and shown as
`dependency of <tool>`. ripeat stops only on a cycle, or when a dependency has no
recipe for this OS. Uninstall refuses to remove something another managed tool
requires ([Install, uninstall, rollback](./install-and-rollback)).

## Versions

`name@version` pins; no version means latest. Where a method cannot pin, ripeat
warns and installs what the source provides. Exact versions across machines are
handled by the lockfile ([The lockfile](./lockfile)).

"Latest" means the newest version **that system's package manager offers**, so it differs
between systems. For example, `node` with no version is the current release on macOS
(Homebrew), an older line on Ubuntu or Fedora, and the LTS release on Windows (that recipe
uses the LTS package). To get the same version everywhere, pin it (`node@24`) where the
package manager can pin, or use a lockfile. A tool that is already installed is never
changed to match a pin: ripeat says so instead ([Risk and `assess`](./assess)).

After an install, ripeat runs the recipe's `version` command (or `check`) and records
the first dotted number in its output. `ripeat list` shows it.

## What `check` does

On Windows, an `installer` option with a `product` ([Vendor installers](./installers)) is also checked without any command: the tool counts as installed when "Add or remove programs" lists a program with that in its name.

If `check` succeeds before ripeat installs anything, the tool counts as
already on the machine: ripeat records it as *pre-existing* and never removes or
upgrades it.
