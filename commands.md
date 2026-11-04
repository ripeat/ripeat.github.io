# Command reference

Every ripeat command, what it changes, and its options. `ripeat` with no arguments, in a terminal, asks what you want to do (create a profile, use a saved one, install by name, or see all commands); without a terminal, or with `ripeat help`, it prints the grouped list. Use `ripeat help install` or `ripeat install --help` for command-specific help; `ripeat help mcp setup` explains assistant setup. Help does not change configuration or contact the network. Commands that change your machine **show what they will do and ask first**;
`--yes` (or `-y`) answers for you, and is required when there is no terminal to ask in.

## Look first: these change nothing

| Command | What it does |
|---|---|
| `ripeat plan [tool[@version] ...]` | The same as `install --dry-run`: what would happen, in order, and why. Takes `--file`, `--profile`, `--scope`, `--no-lock` |
| `ripeat assess [tool[@version] ...]` | How risky the plan is, with reasons ([Risk and `assess`](./assess)). `--json` for programs, `--fail-on medium\|high` to exit with an error at that level, `--accounts <file>` to rate a user policy too, `--file`, `--profile`, `--scope`, `--no-lock` |
| `ripeat find <name>` | Where a tool could be installed from: package managers and GitHub ([Install by name](./find-and-add)). `--no-github` skips GitHub |
| `ripeat list` | The tools ripeat manages, their versions, and whether they were already there |
| `ripeat history` | The last 20 runs and how each ended |
| `ripeat outdated` | Tools with a newer version available ([Versions and upgrades](./upgrades)) |
| `ripeat recipes` | The tools ripeat knows how to install |
| `ripeat recipes check [name ...]` | Asks the real sources whether each recipe still works on this machine ([Recipes](./recipes)) |
| `ripeat rules` | The administrator policy in force here, if any ([Administrator policy](./admin-policy)) |
| `ripeat validate [file ...] [--taps] [--drafts]` | Checks profiles, recipes, policies and lockfiles for mistakes; the flags explicitly check cached packs or waiting drafts ([Checking your files](./validate)) |
| `ripeat schema recipe\|profile\|policy\|admin-policy` | A JSON Schema for editor completion |
| `ripeat version [--check]` | The version. `--check` asks GitHub whether a newer release is out and how to upgrade |

## Change the machine: these ask first

| Command | What it does |
|---|---|
| `ripeat install [tool[@version] ...]` | Shows the plan and its risk, asks, then installs ([Install, uninstall, rollback](./install-and-rollback)). With no names it reads `./ripeat.yaml`. A name with no recipe, at a keyboard, is looked up and offered to you to add |
| | `--file <path>` use that profile · `--profile <name>` use a saved profile (repeatable) · `--dry-run` only show · `--scope user\|system` only methods that do not / do need admin rights · `--no-lock` ignore `ripeat.lock` · `--no-rollback` keep a failed run's partial changes · `--accept-risk` run a HIGH-risk plan without asking · `--yes` answer the question |
| `ripeat uninstall <tool> ...` | Removes tools ripeat installed. Refuses a tool that another managed tool still needs unless `--force`. `--dry-run`, `--yes` |
| `ripeat upgrade [tool ...]` | Upgrades the outdated tools. Cannot be rolled back. `--dry-run`, `--yes` |
| `ripeat rollback` | Undoes the most recent run that is not already undone; run it again for the one before ([Install, uninstall, rollback](./install-and-rollback)). `--dry-run`, `--yes` |
| `ripeat resume` | Carries on an install that paused for a restart ([Restarts and `resume`](./reboots)). `--force`, `--dry-run`, `--accept-risk` |
| `ripeat lock [tool ...]` | Writes `ripeat.lock` with the exact versions installed ([The lockfile](./lockfile)) |
| `ripeat policy` | Users, groups and admin rights from `ripeat.policy.yaml` ([Users and groups](./policy)). A dry run unless `--apply`; `--allow-admin` is needed if the file grants admin rights; `--file <path>` |

## Recipes, profiles and sharing

| Command | What it does |
|---|---|
| `ripeat add <name\|owner/repo>` | Looks a tool up, drafts a recipe, shows it, and saves it if you agree ([Install by name](./find-and-add)). `--from <method>\|github` pick the source · `--as <name>` recipe name · `--bin <program>` the program to put on PATH · `--no-github` · `--refresh` draft one you added again and show the difference · `--manual --check <cmd> --note <text>` record a tool you install by hand ([Recipes](./recipes)) · `--yes` |
| `ripeat add <name> --url <https address> --sha256 <hex>` | Drafts a recipe for a vendor `.dmg`, `.pkg`, `.msi` or `.exe`, after checking the checksum and the vendor's signature ([Vendor installers](./installers)). `--kind`, `--app`, `--signer`, `--product`, `--args`, `--uninstall-args`, `--unsigned` |
| `ripeat bundle` | `export <profile> [file] [--force]`, `import <file-or-https-url> [name] [--sha256 <hex>] [--allow-shell] [--force]`: share a profile together with the recipes you added ([Bundles: profile + recipes](./bundles)). Importing shows and rates every recipe and asks you; there is no `--yes` |
| `ripeat profile` | `create <name> [tool ...] [--no-edit]` (opens your editor), `edit <name>` (opens a saved profile in your editor), `save <name> [file]`, `list`, `show <name>`, `export <name> [file]`, `import <file-or-https-url> [name]`, `remove <name>` ([Profiles: your list](./profiles)) |
| `ripeat tap` | Recipe packs from GitHub or an https address: `list`, `show <name>`, `add owner/repo` or `add https://...` (`--ref`, `--sha256`, `--as`, `--allow-shell`), `update [name]`, `remove <name>` ([Recipe packs (taps)](./taps)). Adding and updating ask you; there is no `--yes` |
| `ripeat drafts` | Recipes an AI assistant prepared: `list`, `show <name>`, `approve <name>`, `reject <name>` ([Drafts from an assistant](./drafts)). Approving asks you; there is no `--yes` |

## AI assistants

| Command | What it does |
|---|---|
| `ripeat mcp` | Runs ripeat as an MCP server over standard input and output, so an assistant can look things up, plan, rate risk and prepare drafts ([ripeat as an MCP server](./mcp)). It is started by the assistant, not by you |
| `ripeat mcp setup [--client claude-code\|claude-desktop\|codex] [--allow-run] [--print] [--yes]` | Connects ripeat to an AI assistant: shows exactly what it will change, keeps a backup of any file it edits, and asks first ([ripeat as an MCP server](./mcp)) |
| `ripeat mcp --allow-run` | Also offers two tools that install or undo, but only after you confirm each run in your own app |

## Environment

| Variable | Meaning |
|---|---|
| `RIPEAT_HOME` | Where ripeat keeps its state, recipes, profiles, drafts and packs. Default: a `ripeat` folder in your user config folder ([Getting started](./getting-started)) |
| `GITHUB_TOKEN` | Optional. Raises GitHub's rate limits for `find`, `add` and `tap`, and lets `tap` read a private repository you can access |
| `NO_COLOR` | Set it to turn off the colours ripeat uses on a terminal |

## Exit status

`0` on success. Anything else means the command failed or was refused (a declined question, a blocked tool, a policy). The message says why, and a command that
stopped before changing anything says so.
