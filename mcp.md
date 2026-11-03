# Using ripeat from an AI assistant: `ripeat mcp`

`ripeat mcp` makes ripeat an **MCP server**, so an AI assistant you already use (Claude, Codex, and others that
speak MCP) can look things up, build a profile, check it, and rate its risk, with ripeat doing the checking. You
keep using your own AI subscription: ripeat needs no API key and makes no model calls.

**The assistant cannot install anything** unless you switch that on yourself ([below](#letting-the-assistant-install-ripeat-mcp---allow-run-off-by-default)), and then it still needs your confirmation for every run. By default it plans; you read the plan and run `ripeat install` yourself.

## Connecting

Want your assistant to do the setup for you? See [Set up ripeat with your AI assistant](./ai-setup), a text you can paste into it.

ripeat itself is the server, so installing ripeat is all that is needed. The easiest way to register it is the setup command:

```bash
ripeat mcp setup                          # lists the assistants it knows
ripeat mcp setup --client claude-code     # or claude-desktop, or codex
```

It uses the full path to ripeat (a path in Homebrew's `bin` is kept, so it survives an upgrade), carries `RIPEAT_HOME` over when it is set in your shell, shows exactly what it will change and asks first (for Claude Desktop's JSON file it shows the new entry and says the rest keeps its value, since the file is rewritten with sorted keys).
`--print` shows the change without making it, `--yes` skips the question, and `--allow-run` also lets the assistant install ([below](#letting-the-assistant-install-ripeat-mcp---allow-run-off-by-default)).
What it does for each assistant:

| Assistant | What `setup` does |
|---|---|
| `claude-code` | Runs `claude mcp add ripeat --scope user -- <path> mcp` after you say yes (the name goes before the options, which `claude mcp add` requires). If a server named `ripeat` exists, it says so and changes nothing |
| `claude-desktop` | Adds `ripeat` under `mcpServers` in Claude Desktop's config file (macOS `~/Library/Application Support/Claude/claude_desktop_config.json`, Windows `%APPDATA%\Claude\...`, Linux `~/.config/Claude/...`). Other settings are kept; the file is rewritten with its keys sorted. A file that is not valid JSON is left alone |
| `codex` | Appends a `[mcp_servers.ripeat]` block to `~/.codex/config.toml`, which the Codex command-line tool and IDE extension read (a hosted Codex with its own connectors does not use this file) (or `$CODEX_HOME/config.toml`) without touching the rest. If the block exists, it changes nothing |

Any file it edits is first copied to `<file>.ripeat-<time>.bak`. To undo, restore that copy (or run `claude mcp remove ripeat` / `codex mcp remove ripeat`). Running it again when ripeat is already set up changes nothing.
Restart the assistant afterwards. By hand, the same thing is:

```bash
claude mcp add ripeat -- ripeat mcp        # Claude Code
codex mcp add ripeat -- ripeat mcp         # Codex
```

For Claude Desktop, add this to its config file (on macOS, `~/Library/Application Support/Claude/claude_desktop_config.json`),
then restart it:

```json
{ "mcpServers": { "ripeat": { "command": "/opt/homebrew/bin/ripeat", "args": ["mcp"] } } }
```

A desktop app has no shell PATH, so give the **full path** (`which ripeat` shows it; on Windows, the shim in `scoop\shims`). A path inside Homebrew's
`bin` or Scoop's `shims` keeps working after an upgrade, whereas a path to a binary you built yourself keeps running that old build. Add
`"env": { "RIPEAT_HOME": "/some/folder" }` only to keep a test away from your own recipes and profiles. To let an assistant install, add
`"--allow-run"` to `args` ([below](#letting-the-assistant-install-ripeat-mcp---allow-run-off-by-default)); a client must be able to show the
confirmation dialog for it to work.

If `ripeat` is not on the PATH the assistant uses, give the full path to it. The exact commands can differ between
versions of an assistant; check its own documentation if one is refused. Run `ripeat mcp` by hand and it just waits for
a client: it speaks the protocol on standard input and output, and prints nothing else.

## What you can ask

Name the tools you want, in your own words: "install lazygit, ripgrep and node 20". The assistant looks each one up,
works out dependencies, writes a profile, checks it, and gives you the plan and the risk. Typical steps:
`ripeat_status`, `ripeat_resolve`, `ripeat_find` (for a tool with no recipe), `ripeat_validate`, `ripeat_plan`,
`ripeat_assess`, and `ripeat_save_profile` if you want to keep it.

## The tools

Each does one job. All are read-only except `ripeat_save_profile`, `ripeat_draft_recipe` and `ripeat_draft_vendor`, which write in ripeat's own folder (and the two run tools, which exist only with `--allow-run`).

| Tool | Answers |
|---|---|
| `ripeat_status` | What machine is this: OS, CPU, which package managers exist, admin rights, whether an administrator policy applies |
| `ripeat_list_recipes` | Which tools does ripeat know, and are they available on this OS? (optional word filter) |
| `ripeat_get_recipe` | What does one recipe look like? |
| `ripeat_list_installed` | What does ripeat manage here, at which versions, and what depends on it? A tool that was on the machine before ripeat comes back with no method and a note that how it was installed is unknown |
| `ripeat_outdated` | Do the managed tools have a newer version? Asks `apt`, `dnf`, `pacman` and Homebrew; upgrading stays with the person (`ripeat upgrade`) |
| `ripeat_version_check` | Compare this server's version with the latest GitHub release; nothing is upgraded |
| `ripeat_check_recipes` | Check up to 30 named recipes against their sources, as `ripeat recipes check` does; skipped options are reported |
| `ripeat_history` | The latest 30 runs (id, command, time, status, steps), to explain what `ripeat rollback` would undo |
| `ripeat_list_taps` | The recipe packs the person added: GitHub repository and commit or HTTPS address and index SHA-256, and whether shell commands are allowed. Adding or updating a pack stays with the person |
| `ripeat_resolve` | For these names: is there a recipe, is it installed, what does it depend on, what would be used, is it blocked? A name with no recipe comes back with the next step |
| `ripeat_find` | Where could an unknown tool come from (package managers, GitHub)? Looks only; descriptions are cleaned and treated as data |
| `ripeat_rules` | What does the administrator allow? ([Administrator policy](./admin-policy)) |
| `ripeat_get_schema` | What shape must a recipe, profile, policy or administrator policy have? |
| `ripeat_validate` | Is this text valid (and is every tool known)? Nothing is saved |
| `ripeat_plan` | What would run, in order, and why is each tool there? |
| `ripeat_assess` | How risky is the plan, with reasons and the exact commands ([Risk and `assess`](./assess)) |
| `ripeat_list_profiles`, `ripeat_get_profile` | The saved profiles ([Profiles: your list](./profiles)) |
| `ripeat_save_profile` | Keep a profile under a name. Checked first; refuses to replace one unless told to. Writes only that file |
| `ripeat_draft_recipe` | For a tool with no recipe: ripeat drafts the recipe itself from a package manager or GitHub release and saves it as a **draft** that does nothing until you approve it ([Drafts from an assistant](./drafts)). Takes a name and an optional source, never recipe text |
| `ripeat_draft_vendor` | Prepare a signed vendor-installer draft from a supplied HTTPS address and published SHA-256. Your client asks before download and inspection; you separately approve the draft in a terminal. Nothing is installed |
| `ripeat_list_drafts`, `ripeat_get_draft` | The drafts waiting for your approval |
| `ripeat_preview_uninstall`, `ripeat_preview_upgrade`, `ripeat_preview_rollback` | Preview those CLI actions without changing anything. Uninstall refuses tools still required by other managed tools; upgrade checks the configured package managers |
| `ripeat_run_install`, `ripeat_run_rollback` | **Only with `--allow-run`.** Install tools, or undo the assistant's last install, after you confirm in your app. Refuse a HIGH-risk plan and any step that needs administrator rights ([below](#letting-the-assistant-install-ripeat-mcp---allow-run-off-by-default)) |

A call with a wrong or missing argument name is answered with the arguments the tool takes (for example "`ripeat_find` has no argument `query`. It takes: `include_github`, `name`"), so an assistant can fix its call in one step.

`ripeat_plan` and `ripeat_assess` take tool names, saved profile names, or profile text, in any mix. They resolve the
plan the way `ripeat plan` does (fallbacks, dependencies, the administrator policy), and accept an optional `scope` of `user` or `system`. They also accept `lockfile` containing the text of `ripeat.lock`; no file is read automatically because the assistant's working folder is unknown. Supply the same lockfile and scope to `ripeat_plan`, `ripeat_assess`, and `ripeat_run_install` so they agree. The same lockfile rules and limitations as the CLI apply ([The lockfile](./lockfile)).

A tool that is already on the machine comes back with no method and no risk level: ripeat does not know how it got there
(it may come from a version manager, not the package manager the recipe uses) and will not change it. `ripeat_status` says
whether the ripeat *process* runs as root or administrator (`ripeat_running_as_admin`), which does not mean the person
cannot get admin rights, for example through `sudo`.

A tool that is already installed in a different version than was asked for comes back with a `warning` (and the installed
version), so the assistant can say plainly that, for example, node 24 is installed and node 20 will not be.

Recipe packs are the person's alone: the assistant is told never to run `ripeat tap add` or `ripeat tap update` ([Recipe packs (taps)](./taps)).

A tool that has no recipe is not created by the assistant. It finds where the tool comes from, asks ripeat to prepare a draft
(`ripeat_draft_recipe`), and tells you to read it and run `ripeat drafts approve <name>` in a terminal, which asks you
([Drafts from an assistant](./drafts)). A vendor `.dmg`, `.pkg`, `.msi` or `.exe` can instead be prepared with `ripeat_draft_vendor` after your confirmation. Unsigned installers, hand-installed (`manual`) tools and shell recipes are not drafted through MCP. HIGH-risk plans and admin-requiring installs remain refused by the run tools.

## Letting the assistant install: `ripeat mcp --allow-run` (off by default)

By default the assistant can only plan. If you want it to be able to install for you, register the server with a flag you set yourself:

```bash
claude mcp add --scope user ripeat -- ripeat mcp --allow-run
```

That adds two tools, `ripeat_run_install` and `ripeat_run_rollback`. The assistant cannot turn them on; only the registration does, and an
administrator policy with `deny_mcp_run: true` ([Administrator policy](./admin-policy)) switches them off whatever the registration says.

Even then nothing runs on the assistant's word:

- **You confirm every run, outside the model.** ripeat asks your app to show a confirmation dialog (MCP "elicitation") listing the tools, the exact
  commands and the risk, with one yes/no box that must be ticked; accepting the form with the box unticked counts as no. Some apps show only the first lines of the
  question (Claude Code cuts it after two), so the question is two lines: the exact commands first, then the risk and how to undo. The assistant cannot answer it. If your app cannot show such a dialog, or you decline, nothing is run and the assistant is
  told to give you the command instead. Claude Code shows the dialog; check yours before relying on this.
- **Narrow reach.** A plan rated HIGH, a blocked tool, and any step that needs administrator rights are refused without asking, because a password
  cannot be typed over this channel; you run those yourself. `uninstall`, `upgrade` and `policy` are not offered.
- **Rollback is limited to its own runs.** `ripeat_run_rollback` undoes the most recent run only if it was started through these tools (the
  journal says `mcp: install ...`), also after you confirm. A run you made in your terminal is not the assistant's to undo.
- **The same rules as the terminal.** A failed install is undone automatically, the administrator policy applies, and `ripeat history` and
  `ripeat rollback` see these runs like any other.
- **The state lock is held only during a run,** so your own `ripeat` commands work while the assistant is connected. Programs ripeat starts get no
  input, because the server's input is the protocol. With the run tools on, the server offers protocol revisions up to 2025-11-25.

`ripeat_run_install` takes the same input as `ripeat_plan`. The assistant is told to plan and show the risk first, and to stop if you decline.

## Prompts

Prompts are ready-made requests you pick in your assistant (often as slash commands). Each wraps your words in a marked
block, with the steps and the rules, so the result is consistent.

| Prompt | For |
|---|---|
| `build-profile` | Turn the tools you name (and any notes, like "no admin rights") into a checked, risk-rated profile |
| `add-tool` | Find where a tool comes from and say how to create its recipe |
| `review-plan` | Check a saved or pasted profile and explain in plain words whether it is safe to run |
| `explain-failure` | Explain an error ripeat printed and what to try ([When something fails](./failures)) |
| `uninstall-safely` | Check what depends on tools before you remove them |

## Resources

Reference material the assistant can read: the JSON Schemas (`ripeat://schema/recipe`, `profile`, `policy`,
`admin-policy`), selected ripeat documentation pages (`ripeat://docs/<page>`, built into the program so they work offline),
worked examples (`ripeat://examples/profile`, `ripeat://examples/recipe`), the administrator rules (`ripeat://rules`),
and any recipe or saved profile by name (`ripeat://recipes/<name>`, `ripeat://profiles/<name>`).

The server also tells the assistant, when it connects, the usual order of the tools and its rules. Assistants differ in how
much of MCP they support: tools work almost everywhere, while prompts and resources are shown differently, or not at all,
in some. The tools and that built-in guidance are enough on their own.

## Safety

- **Plan-first by default.** Saving a profile or draft writes a file in ripeat's own folder. Only the optional run tools can install or roll back, with the limits and confirmation described above. Upgrade and uninstall are not offered as run tools.
- **You do not have to say "don't install".** The server hands every assistant standing instructions: a request to set up,
  add, remove or update tools means "prepare the plan, show the risk, give me the command". The same boundaries are repeated
  in every prompt.
- **Boundaries the assistant is told to keep** (an assistant with a shell is a separate program, so these are instructions, not
  locks; the locks are below): never run ripeat itself in a shell to change the machine; never add `--yes`, `--accept-risk` or
  `--allow-admin` to a command it hands you; explain a HIGH or BLOCKED result and offer a safer choice instead of working around
  it; never edit or weaken the administrator policy file; never write shell commands or recipes.
- **The locks that do not depend on the assistant.** `install`, `uninstall` and `upgrade` ask before acting and refuse when
  nothing can answer (no terminal) unless `--yes` is given; a HIGH-risk plan also needs `--accept-risk`, which `--yes` does not
  replace; the administrator policy blocks what it forbids whatever flags are used; every run can be undone with `ripeat rollback`.
  If you want to stop an assistant running installs on its own, do not let it run shell commands without asking (in Claude Code,
  keep `Bash(ripeat install:*)` out of the allowed list; in Codex, keep approvals on).
- **Your words cannot rewrite the rules.** Prompts put what you type in a marked block, and a closing marker typed inside it
  is neutralised.
- **Outside text is data.** Descriptions that come from package managers or GitHub are stripped of control characters, cut
  to a length, and described to the assistant as data, never instructions.
- **The administrator policy applies.** What it forbids is reported as blocked, and no argument overrides it.
- **A broken administrator policy stops lookups that depend on it** rather than being ignored ([Administrator policy](./admin-policy)).
- A saved profile only names tools, and running it still goes through the risk gate: a HIGH-risk plan needs your explicit say-so
  at the command line ([Risk and `assess`](./assess)).

## Limits

- The assistant cannot create recipes (it can ask ripeat to prepare a draft that you approve). Unless you started the server with `--allow-run`, it cannot run installs either.
- It talks over standard input and output only (no web server), one client at a time.
- Whether an assistant uses the tools well depends on the assistant. Check the plan and risk it reports, and read them
  yourself before running anything.
- `ripeat_find` searches GitHub, so it is subject to GitHub's rate limits (set `GITHUB_TOKEN` to raise them).
- ripeat decides whether a tool is "already on the machine" with the `PATH` it was started with, and an assistant starts `ripeat mcp`
  with its own `PATH`. An assistant that adds a folder to it (Codex adds its bundled `rg`) can make ripeat report a tool as present
  that is missing in your normal terminal. `ripeat plan` in your own terminal is the answer that counts.
