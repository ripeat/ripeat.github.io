# Checking your files: `validate` and `schema`

## `ripeat validate [file ...]`

Checks ripeat's input files and reports **every** problem in one go, before anything
is installed. Nothing is read from or written to the machine's state.

```bash
ripeat validate                    # ./ripeat.yaml, ./ripeat.policy.yaml, ./ripeat.lock, your recipes
ripeat validate my-recipe.yaml     # just this file
ripeat validate --taps            # cached recipes in added packs
ripeat validate --drafts          # waiting drafts, including approval checks
ripeat validate --taps --drafts   # both groups, reported separately
```

The kind of file is recognised from its content: `*.lock` is a lockfile, otherwise
`platforms:` means a recipe, `tools:` a profile, `groups:` or `users:` an account policy, and keys such as
`allow_methods:` or `deny_tools:` an administrator policy.
Any other file is an error. With no arguments, the files above are checked if they
exist, the system administrator policy ([Administrator policy](./admin-policy)) if there is one, every saved
profile ([Profiles: your list](./profiles)), plus every `*.yaml` in `$RIPEAT_HOME/recipes/`.

Each file gets one line: `✓` valid, `!` valid with warnings, `✗` has errors, with each problem
under it as `error: ...` or `warning: ...`. Errors would make ripeat fail, and the command then
exits with an error. Warnings work but are probably not what you meant. With no errors it ends
with `All N files valid.` (`, with M warnings` when there are some). Marks are coloured on a
terminal only; `NO_COLOR` turns the colour off.

### Errors in every file kind

Unknown keys (a typo such as `pakage:` or `tool:`) and invalid YAML.

### Recipes ([Recipes](./recipes))

| Error | Warning |
|---|---|
| name not lowercase letters, digits, `.` `_` `-` | no `check` (ripeat cannot tell if the tool is already there) |
| no `platforms`, a platform other than `linux`/`windows`/`darwin`, a platform with no options | `shell` without `uninstall` (cannot be uninstalled or rolled back) |
| unknown install method | `deb_url` (not checksum-verified) |
| a key the method requires is missing (`package`; `url`, `install`, `repo`, `asset`, `bins`, `sha256` per method) | a URL using plain `http` |
| `scope` other than `user` or `system`; `source` other than `winget` or `msstore` | a key the method does not use |
| `requires` a tool that has no recipe, or a dependency cycle | `github_release` without `checksum_asset` |
| `repo` not `owner/name`; `asset` not a valid regular expression; unknown `{placeholder}`; `sha256` not 64 hex characters | |
| `product` that is blank, longer than 80 characters or on several lines | `product` on a dmg (it is only read on Windows) |
| `package` of a package-manager method that is not plain package names (letters, digits and `. _ + @ : / -`, never starting with `-`): the names become arguments of the package manager, and `-o ...` after one would be read as an option | |

### Profiles ([Profiles: your list](./profiles))

Wrong `version`, empty list, and any tool that has no recipe.

### Policies ([Users and groups](./policy))

Unsafe or invalid names, duplicate users, a shell that is not an absolute path, an empty policy.

### Administrator policies ([Administrator policy](./admin-policy))

Unknown keys, a version other than `1`, an unknown install method, a malformed domain, and a method
that is both allowed and denied. A tool name with no recipe is a warning. For the system policy
file, `validate` also reports what ripeat itself would refuse: a file not owned by root or writable by others.

### Lockfiles ([The lockfile](./lockfile))

Unknown method, a malformed platform key (not like `linux/amd64`), a `sha256` that is not
64 hex characters. A pin for a tool with no recipe is a warning.

## What it does not check

`ripeat validate` with no arguments checks your own files: `ripeat.yaml`, `ripeat.policy.yaml`, `ripeat.lock`, your recipes, your saved profiles and the
administrator policy. It does not check added packs ([Recipe packs (taps)](./taps)) or waiting drafts ([Drafts from an assistant](./drafts)) unless `--taps` or `--drafts` is selected. With those flags it checks the selected groups and any explicitly named files, not the default file set. Groups are reported separately and errors give a nonzero exit status. Pack checks use cached files and pack rules, not a network refresh; draft checks use the same review as approval. Name a file to check its format by hand: `ripeat validate path/to/draft.yaml`.

## Strict recipes

Recipes are read strictly everywhere, not only by `validate`: an unknown key in a
recipe or in one of its platform entries is an error for every command that loads
recipes. Fix it, or find it with `ripeat validate`.

## `ripeat schema recipe|profile|policy|admin-policy`

Prints a JSON Schema for that kind of file, so an editor can complete keys and
underline mistakes while you type. It is generated from the same types ripeat reads,
so it lists exactly the keys and install methods ripeat knows.

```bash
ripeat schema recipe > recipe.schema.json
```

With the YAML extension for VS Code, point a file at it with a first line such as
`# yaml-language-server: $schema=./recipe.schema.json`.

The schema checks structure only. `ripeat validate` is the authority: it also checks
things a schema cannot, such as whether a required recipe exists or whether
dependencies loop.
