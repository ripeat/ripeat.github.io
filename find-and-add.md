# Install by name: `find` and `add`

For a tool that has no recipe yet. ripeat looks the name up, drafts an ordinary recipe
([Recipes](./recipes)), shows it to you, and saves it only if you agree. There is no
AI in this: it is a fixed set of lookups and rules, so the same name gives the same answer.
Installing is a separate, normal step afterwards.

## `ripeat find <name>`

Looks the tool up and lists where it could be installed from. Nothing changes.

```bash
ripeat find ripgrep          # a plain name
ripeat find BurntSushi/ripgrep   # a GitHub project, as owner/repo
```

Sources searched, in this order: the package managers installed on this machine
(`apt`, `dnf`, `pacman`, `brew` and `brew_cask`, `winget`, `choco`, `scoop`, `snap`, `flatpak`), then GitHub
(unless you pass `--no-github`). Each result shows the install method, the package or
project, its version and description. Results the administrator policy forbids are shown
as blocked, and GitHub projects that cannot be used are listed with the reason.

- A package manager lists a tool only if its name matches **exactly**. (`winget search --exact` also matches tags, so ripeat
  keeps only results whose id, display name or last part of the id is the name: `bat` finds `sharkdp.bat`, not every package tagged "bat".) Flatpak is the one exception: Flathub names apps by
  id (`org.mozilla.firefox`), so `firefox` matches an app whose id ends in `.firefox` or whose display name is Firefox
  (at most three are shown). An exact id (`org.gnome.Calculator`) is always found; a plain name is searched in Flathub's
  catalog data, which flatpak fetches for you on its own schedule and can be missing on a machine where Flathub was only just added
  (`flatpak update --appstream` fetches it).
- **`winget` searches two catalogs:** the community one, and the **Microsoft Store**, which holds apps the community
  catalog does not (ChatGPT, for example). A Store result is marked "Microsoft Store" and becomes a recipe option with
  `source: msstore` ([Recipes](./recipes)). The community catalog is listed first, so it wins when both have the name.
- **`scoop`** searches the buckets you have added. An app outside the default `main` bucket is named `bucket/app`
  (`extras/firefox`); scoop cannot install it unless that bucket is added, which is why only buckets you have added are searched.
- **`snap`:** a snap that needs "classic" confinement (VS Code is one) is listed as not usable, because ripeat installs
  with plain `snap install`, which refuses it.
- On GitHub, a plain name must match a repository name exactly (most starred first, at most
  three tried); `owner/repo` goes straight to that project.
- A GitHub project is a candidate only if its latest release has a file for this OS and CPU
  **and** a published SHA-256. Otherwise it is listed as not usable, because ripeat will not
  install what it cannot verify.

### When nothing is found

`find`, `add` and the offer made by `install` all end the same way: what was searched, anything found but refused (blocked by the
administrator policy, or not usable, each with its reason), and what to do next:

- try the exact name or another spelling;
- if the project is on GitHub, `ripeat add owner/repo` (it needs a release file for this system and a published SHA-256);
- if the vendor offers a `.dmg`, `.pkg`, `.msi` or `.exe`, with a published SHA-256 (an `.exe` also needs `--product` and `--args`): `ripeat add <name> --url <address> --sha256 <checksum>`
  ([Vendor installers](./installers)); for an archive (`.zip`, `.tar.gz`), write a recipe with method `url` ([Downloads (GitHub, URL)](./downloads))
  or ask an AI assistant to draft one, and `ripeat validate` it;
- if you will install it yourself (or already have), record that: `ripeat add <name> --manual --check "<command>" --note "<how to get it>"`, so ripeat can tell when it is there;
- some apps cannot be installed this way: ones that come with the system (Safari), ones that need a paid licence
  (Microsoft 365), and ones whose installer writes to system areas or runs services (ripeat refuses those `.pkg` files).
  Install those yourself and record them with `--manual`; ripeat leaves anything already on the machine alone.

Names are limited to letters, digits and `. _ + -` (or `owner/repo`), so a name can never be
mistaken for an option or a command.

## `ripeat add <name> [--from method|github] [--as recipe] [--bin program] [--refresh] [--url address --sha256 hex] [--yes]`

1. Looks the tool up as above, and drops anything the administrator policy forbids.
2. Takes the **first** candidate (package managers come before GitHub), or the one you pick with
   `--from` (an install method such as `apt`, or `github`). `ripeat find` shows the others.
3. For a GitHub project, asks before downloading the release file, then downloads it, checks its
   SHA-256, and looks inside to find the program to put on `PATH`. If several programs are inside
   and none is clearly the project's own, it stops and asks for `--bin`.
4. Drafts the recipe, checks it with the same rules as `ripeat validate` and the administrator
   policy, and prints it.
5. Asks "Save this recipe?" and, on yes, writes `$RIPEAT_HOME/recipes/<name>.yaml`.

Then install it the usual way: `ripeat install <name>`.

| Flag | Meaning |
|---|---|
| `--from` | Use this source: an install method (`apt`, `brew`, ...) or `github` |
| `--as` | Name for the recipe. Default: the tool's name, in lowercase. Refuses a name that already exists, built-in ones included |
| `--bin` | The program to put on `PATH`, when ripeat cannot tell. Skips opening the download |
| `--refresh` | Draft a recipe **you added** again and replace it if you agree: see below |
| `--yes` | Do not ask. Without it, ripeat needs a terminal to answer in, and refuses otherwise |
| `--no-github` | Do not look on GitHub |

### What the drafts look like

- **A package manager:** one option for this OS (for example `{method: apt, package: ripgrep}`),
  plus a `check` command that asks that package manager whether the package is installed.
  Homebrew drafts are marked `scope: user`.
- **A GitHub release:** a `github_release` option ([Downloads (GitHub, URL)](./downloads)) with
  `scope: user`, an `asset` pattern that keeps matching new versions of the same file (the version
  number becomes a pattern, and the OS and CPU become `{os}` and `{arch}`-style placeholders where the
  file name spells them that way), a `checksum_asset` if the checksum is a separate file, and the
  program found inside.

A draft covers **this OS only**, and for GitHub only this CPU's naming. To support more systems, add
sections for them by hand ([Recipes](./recipes)); the file is a normal recipe with a comment at the top.

When several release files look alike for this machine (for example two builds that rank the same),
ripeat does not guess and the project is listed as not usable. A project whose `musl` or `msvc` build exists
is preferred over its alternatives.

### A tool you install by hand: `--manual`

```bash
ripeat add excel --manual --check 'test -d "/Applications/Microsoft Excel.app"' --note "Install Microsoft 365 (it needs a licence)."
```

Drafts a `manual` recipe ([Recipes](./recipes#manual-a-tool-you-install-by-hand)): no lookup and no download, just the command that tells when the tool is installed and one line saying how to get it.
At a terminal, a missing `--check` or `--note` is asked for. The draft is shown and saved only if you agree, like any other.

### From a vendor's download address: `--url` and `--sha256`

For an app offered only as a `.dmg`, `.pkg`, `.msi` or `.exe`, `ripeat add <name> --url <https-address> --sha256 <the vendor's checksum>` drafts an
`installer` recipe: it checks the file against your checksum, reads who signed it, pins that signer, and shows the draft
([Vendor installers](./installers)). Other flags: `--kind`, `--app`, `--signer`, `--unsigned`.

### Creating the recipe while installing

`ripeat install opera` for a name with no recipe offers to do all of the above on the spot ([Install, uninstall, rollback](./install-and-rollback)). With **several** names that have no recipe
(`ripeat install opera ollama chatgpt`) it asks once, "Look for all 3 and draft recipes?", prepares every draft (asking which source when one is found in more than one place), prints
**all of them in full**, and asks once, "Save these 3 recipes?". A tool that cannot be drafted does not stop the others: the good drafts are saved on your yes, and the install waits until
the rest have recipes too. For one name it is the single flow below:
it asks "Look for it and draft a recipe?", shows what it found (when there are several sources it numbers them and asks which, Enter
for the first, `n` to skip), shows the draft, saves it on a yes, and goes on to the plan and the usual install question. It asks at a
keyboard only. Nothing is offered with `--dry-run`, in a script or from an AI assistant, and `--yes` does not answer it.
`owner/repo` names are for `ripeat add`, not offered here.

### Refreshing a recipe: `--refresh`

A package gets renamed, a project renames its release files. When `ripeat recipes check` ([Recipes](./recipes)) says a recipe
you added no longer works, draft it again:

```bash
ripeat add lazygit --refresh
```

- It looks the tool up again and prefers **the source the recipe already uses**, so the diff shows what changed and not a
  different source. If that source no longer works it says so and uses another.
- It prints what changes (`-` lines go away, `+` lines are new, a line or two around each change). If nothing changes, it says
  "Already up to date" and stops.
- It asks "Replace the recipe with the new draft?" and, on yes, saves the old file next to it as `<name>.yaml.old` and writes
  the new one. To go back, rename the `.old` file.
- A draft knows only this machine's OS, so a refresh **keeps the sections for other systems, `requires` and `version`** exactly as
  they were. It does rewrite the file in the usual drafted layout, so the diff also shows reformatting.
- It works on recipes you added only. A built-in recipe is updated by upgrading ripeat (`ripeat version --check`, [Versions and upgrades](./upgrades)).
  Without `--refresh`, `add` still refuses a name that exists.

## Limits

- The program inside a GitHub download is found by looking at the file, not by running it.
- Only the latest release is looked at; pinning an older one is done in a recipe or with `name@version` later.
- Package-manager lookups do not search by description or similar names: the name must be exact.
- A draft is a starting point, so read it before installing.
