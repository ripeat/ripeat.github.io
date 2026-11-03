# Recipe packs: `ripeat tap`

A **tap** is a GitHub repository of recipes that someone maintains: a team's approved tools, a community collection. You add it once, it is
pinned to one commit, and its recipes then work with `ripeat install` like any other. ripeat does not run a server or own a catalog; a pack is just
a repository you chose.

```bash
ripeat tap add acme/dev-tools            # show what is in it, ask, pin it
ripeat tap add https://example.org/pack.yaml   # a pack on any static host, pinned by SHA-256
ripeat tap                               # the packs you have, their commit, how many recipes
ripeat tap show dev-tools                # the recipes of one pack
ripeat tap update                        # look for newer commits; show exactly what changes; ask
ripeat tap remove dev-tools              # forget it and delete its recipes
```

## What a pack looks like

A repository with a `recipes/` folder at its top, one `name.yaml` per recipe, in the ordinary recipe format ([Recipes](./recipes)). The file name
must match the recipe's `name`. Sub-folders and other files are ignored. A pack may hold at most 200 recipes of at most 64 KB each.

## A pack at an https address

Any static host (GitLab, Codeberg, your own site) can serve a pack as **one YAML file**, the pack index:

```yaml
ripeat_pack: 1
name: acme-web          # optional: the default name for the pack
recipes:
  - name: lazygit
    check: lazygit --version
    platforms:
      linux: {method: apt, package: lazygit}
```

`ripeat tap add https://example.org/pack.yaml [--sha256 hex] [--as name]` downloads it (https only, redirects to http refused, at most 512 KB, at most 200 recipes), prints its
SHA-256, shows each recipe and asks, like any pack. The SHA-256 of the file is the pin, shown by `ripeat tap` where a commit would be. If the owner of the pack
publishes the SHA-256, pass it as `--sha256` and the add stops when the file differs. `ripeat tap update` downloads the file again and, when its SHA-256 changed,
shows the recipes added, changed and removed and asks. Every rule below applies unchanged; `--ref` does not apply to an address.

## Adding one

`ripeat tap add owner/repo [--ref branch-or-tag] [--as name] [--allow-shell]` downloads the pack at the commit the branch (default: the repository's
default branch) points to **now**, checks every recipe, and shows you the result before anything is saved:

```
acme/dev-tools at commit 0123456 (main). It holds 3 recipe files:
  git              not loaded: a built-in recipe has this name, and a pack cannot replace it
  lazygit          brew            risk LOW
  shelly           not loaded: it runs a shell command, which a pack may only do if you add it with --allow-shell
Add the pack "dev-tools" (pinned to 0123456)? [y/N]
```

There is **no `--yes`**, and without a terminal it refuses: adding a pack changes which recipes ripeat trusts, so only a person answers. An
administrator policy with `deny_taps: true` ([Administrator policy](./admin-policy)) forbids adding or updating packs and hides the recipes of packs
already added.

## What a pack can and cannot do

- **Strictly checked.** Each recipe must parse (unknown keys are errors) and pass the same checks as `ripeat validate`. One that does not is listed as
  "not loaded" with the reason, and the others still load.
- **Plain package names.** A `package` must be plain names (no spaces-then-options such as `x -o ...`), because the names become arguments of `apt`, `brew` and the rest.
  This applies to every recipe, not only a pack's.
- **Plain probes only.** ripeat runs a recipe's `check` and `version` commands through the shell every time it plans, to see whether a tool is already installed, so even
  `ripeat plan` would run whatever a pack put there. A pack's `check` and `version` may therefore only be a tool's own name with flags (`lazygit --version`) or a package
  manager's list or info command (`brew list --formula lazygit`, `winget list --id Git.Git --exact`, `dpkg -s x`, `command -v x`, `test -d <path>`). No `;`, `&`, `|`, `<`, `>`, `$`,
  backticks or parentheses, and no other program. `tap add` and `tap update` **show each recipe's check command**, and the rule is applied again every time recipes are loaded.
  Adding the pack with `--allow-shell` lifts it, together with the shell-recipe rule.
- **No shell commands, unless you say so.** A recipe with a `shell` option is not loaded unless you added the pack with `--allow-shell`, which prints a
  warning. An `installer` recipe marked `unsigned: true` is never loaded from a pack. Nor is an `.exe` or `.pkg` `installer` (it would run vendor code as administrator on the pack author's say-so).
- **Cannot replace what is yours.** The layers, lowest first: the built-in recipes, then the packs, then your own recipes. A pack **cannot** take a
  built-in recipe's name (so a pack cannot swap in its own `git`), and your own recipe with the same name wins over a pack's. If two packs carry the
  same name, the pack that sorts first by name keeps it, and `ripeat tap` lists the other as not loaded.
- **Rated as strictly as your own.** A recipe from a pack counts as one you added: a download, an installer or a shell command in it is rated
  accordingly ([Risk and `assess`](./assess)), and the reason names the pack. `ripeat recipes check` shows its origin as `tap:<name>`.
- **Checked again every time it loads.** The shell and unsigned rules are applied when the recipes are loaded, not only when the pack is added, so a
  file dropped into the pack's folder by hand is not loaded either.

## Updating

Nothing changes on its own. `ripeat tap update [name]` fetches the commit the pack's branch points to now. If it is the one you have, it says so. If
not, it shows each recipe that is **added, changed or removed**, with a line-by-line difference for the changed ones, and every recipe that would not be
loaded, then asks. On yes, the folder is replaced and the pin moves; on no, the pack stays exactly as it was. Tools already installed from a pack keep
working after an update or `remove`; a recipe that disappears only stops being available for new installs.

If a recipe from a pack stops working, `ripeat recipes check` says so and points at the right fix for a pack: `ripeat tap update`, the pack's maintainer, or your
own recipe with the same name, which wins ([Recipes](./recipes)). `ripeat add <name> --refresh` does not apply to a pack's recipe.

## Where it lives

`$RIPEAT_HOME/taps/<name>/` holds the accepted recipes and `$RIPEAT_HOME/taps.json` records each pack: repository or address, branch, pinned commit and
whether shell recipes are allowed. Downloads use the GitHub API (a rate limit applies; `GITHUB_TOKEN` raises it).

## Limits

- GitHub repositories (public ones; with `GITHUB_TOKEN`, private ones you can read) and https addresses of a pack index. There is no signature on a pack: its trust is the commit you
  saw and approved, which is why updates show a diff.
- The commit is pinned, but ripeat does not verify who wrote it. Add packs from people you trust, and read what `tap add` shows.
- An assistant connected through `ripeat mcp` is told never to run `ripeat tap add` or `ripeat tap update`.
