# Recipes prepared by an AI assistant: `ripeat drafts`

An AI assistant connected through `ripeat mcp` ([ripeat as an MCP server](./mcp)) can **prepare** a recipe for a tool that has none. It cannot
**create** one. The recipe is yours only after you read it and approve it in a terminal.

## How it works

1. The assistant calls `ripeat_draft_recipe` with a tool's name (and optionally a source such as `apt`, `brew` or `github`). For a vendor installer it can call `ripeat_draft_vendor` with the vendor's HTTPS address and published SHA-256. Your app must show a confirmation before ripeat downloads or inspects it.
2. **ripeat writes the recipe, not the assistant.** For package-manager and GitHub requests it looks the tool up and drafts it with the same fixed rules as `ripeat add` ([Install by name](./find-and-add)). For a vendor request it inspects the confirmed download and checks its checksum and signature. It checks the recipe and
   saves it in `$RIPEAT_HOME/drafts/<name>.yaml`. The tool accepts a name, a source and a recipe name, and no recipe text, so the assistant
   has no way to put its own commands into a recipe.
   The result says where the file is (`saved_to`), and `ripeat_list_drafts` says which folder it looked in.
3. **A draft does nothing.** ripeat never reads the drafts folder as recipes: `ripeat plan` and `ripeat install` still say there is no
   recipe for the name.
4. You review and approve it yourself:

```bash
ripeat drafts                     # the drafts waiting, with install method and risk
ripeat drafts show lazygit        # the recipe, its risk and the reasons
ripeat drafts approve lazygit     # asks "Approve ... ? [y/N]"; on yes it becomes a recipe
ripeat drafts reject lazygit      # delete the draft
```

`approve` checks the draft again from the file as it is now (it may have been edited), shows the recipe and its risk, and asks.
**There is no `--yes`**, and without a terminal it refuses, so only a person can answer. On a yes the file moves into
`$RIPEAT_HOME/recipes/`, marked as approved, and `ripeat install <name>` works.

## What a draft can be

- **Package-manager, GitHub-release and signed vendor-installer sources.** The vendor tool supports `.dmg` and `.pkg` on macOS, `.msi` and `.exe` on Windows. It requires the vendor's published checksum and verifies and pins the signer; unsigned installers are refused. An `.exe` also needs the product name and vendor-documented silent arguments. It never runs the installer. `shell` and `manual` drafts are refused at approval.
- **Not over an existing recipe.** A name that already has a recipe is not drafted, and approval refuses to replace one.
- **Within the administrator policy.** A source the policy forbids is not drafted ([Administrator policy](./admin-policy)), and approval
  checks again.
- **A GitHub source downloads** the release file to a temporary folder to find the program inside it. Its SHA-256 is checked, nothing
  in it is run, and the folder is deleted afterwards.

## What the assistant also gets

`ripeat_list_drafts` and `ripeat_get_draft` read the waiting drafts (text, method, risk, any problem). It is told to hand you the
`ripeat drafts approve <name>` command and never to run it.

## Limits

- The risk shown is the one `ripeat assess` would give the recipe once it is yours (a recipe in your folder counts as one you added).
- A draft that does not fit the current machine (no option for this OS) cannot be approved here.
- Approving does not install anything. Installing is the usual `ripeat install`, with its own plan, risk rating and question.
