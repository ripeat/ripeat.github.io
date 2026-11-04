# Profiles: `ripeat.yaml`

A profile is the list of tools you want. It says *what*, never *how* (that is the
recipe's job, see [Recipes](./recipes)).

```yaml
version: 1
tools:
  - git
  - node@20
  - typescript
```

- `version` must be `1`.
- Each entry is `name` or `name@version`. Without a version ripeat installs the latest. A pinned version is only
  installed if the tool is not already on the machine: an existing tool is never changed, and ripeat warns when the
  version it has is not the one you asked for ([Risk and `assess`](./assess)).
- Unknown keys are errors, so a typo such as `tool:` fails loudly instead of being ignored.
- An empty list is an error.

## Which file is used

| You run | ripeat uses |
|---|---|
| `ripeat install` | `./ripeat.yaml`, if it exists |
| `ripeat install --file other.yaml` | that file |
| `ripeat install git` | only `git`; `./ripeat.yaml` is not read |
| `ripeat install --file p.yaml git` | the file's tools plus `git` |

`ripeat validate` ([Checking your files](./validate)) also checks that every tool has a recipe.

If there are no tool names and no profile, `install` stops with an explanation.

## Saved profiles: run a plan by name

A profile you keep using can be saved under a name, so you never have to pass a path.

```bash
ripeat profile create web git node ripgrep   # a new profile: opens your editor with these tools listed
ripeat profile edit web                       # change a saved profile in your editor
ripeat profile save web ripeat.yaml    # keep ./ripeat.yaml as "web" (the file defaults to ./ripeat.yaml)
ripeat profile list                    # the saved profiles and their tools
ripeat profile show web                # print one
ripeat profile remove web              # delete one
ripeat profile export web web.yaml     # write it to a file to share (without a file it prints it)
ripeat profile import web.yaml         # save a shared file as a profile (a file or an https:// address)

ripeat profile create web git node --no-edit  # save the listed tools straight away, no editor

ripeat install --profile web                      # run it
ripeat install --profile web --profile data       # run several together
ripeat plan --profile web                         # preview, as usual
ripeat assess --profile web                       # rate its risk ([Risk and `assess`](./assess))
```

- **Writing one in your editor.** `ripeat profile create <name> [tool ...]` opens a new profile in your editor (`$VISUAL`, then `$EDITOR`, then `vi`, or `notepad` on Windows; a value such as `code --wait` works, a path containing a space does not). The tools you list are already in it, with a short comment on the format; leave the editor to save. `ripeat profile edit <name>` opens a saved one. When you close the editor ripeat checks the text exactly as `profile save` does (a typo such as `tool:` is an error, not silently ignored). If it is not valid it shows the error and asks whether to open the editor again; if you say no, nothing is saved and your text stays in the temporary file it names. An edit that changes nothing saves nothing. `create` refuses a name that already exists and `edit` one that does not, and each says which command to use. Both need a terminal; without one, `create <name> tool ... --no-edit` saves the listed tools directly.
- **Finding your next step:** `ripeat profile list` shows saved profiles and their tools. After saving one, ripeat suggests assessing it before installation. `ripeat profile show` and export without a file still print the original YAML.
- **Where:** `$RIPEAT_HOME/profiles/<name>.yaml`, in exactly the format above, so you can also edit them by hand.
- **Names:** lower-case letters, digits, `.` `_` `-`, up to 64 characters; never a path.
- **Saving checks the file** (the same rules as above) and refuses to replace an existing profile unless you add `--force`.
  It notes any tool that has no recipe yet, but still saves.
- **Combining:** `--profile` can be repeated and mixed with `--file` and tool names. A tool asked for twice is
  installed once. If two ask for different versions (`node@20` and `node@18`) ripeat stops and says so; a pinned
  version wins over "latest".
- A profile only names tools, so it works on any system; each machine uses its own recipes.
- `ripeat validate` also checks every saved profile ([Checking your files](./validate)).

## Sharing a profile

```bash
ripeat profile export web web.yaml                                   # you: write it out
ripeat profile import web.yaml                                       # a colleague: from the file
ripeat profile import https://example.com/team/backend.yaml          # or from an address
ripeat profile import https://example.com/ripeat.yaml backend        # a generic file name needs a name
```

- **Need to share recipes too?** Use a [bundle](./bundles).
- **A profile carries no commands.** It names tools and versions, nothing else, so importing one runs nothing and installs
  nothing. What it would do on your machine is decided by your own recipes, the plan, the risk rating and the confirmation
  question, exactly as for a profile you wrote yourself. Still run `ripeat assess --profile <name>` before installing a profile
  from someone else.
- **The name** comes from the file name (`web.yaml` becomes `web`); `ripeat.yaml` says nothing, so give a name. An existing
  profile is replaced only with `--force`, and an export refuses to overwrite a file without `--force`.
- **Addresses** must be `https://`. Plain `http://` is refused, and so is a redirect to it. The download times out after 20 seconds
  and a file over 64 KB is refused.
- **Checked on the way in:** the file must be a valid profile (the rules above). After saving, ripeat lists the tools and marks
  those with no recipe on this machine; the profile can still be saved, but cannot run until `ripeat add <name>` provides one.
- **Recipes are not shared.** A recipe you added yourself stays on your machine, because a recipe can hold commands and should
  only come from someone you chose to trust. A teammate creates the same tool with `ripeat add` and reviews it themselves.
