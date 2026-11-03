# For agents: the skill and `llms.txt`

Two files help an AI agent use ripeat safely when it only has a shell, or when it reads the web.

## The skill

A skill is a short instruction file (`SKILL.md`) that agent tools such as Claude Code load on demand. ripeat's skill can guide an agent through installing ripeat and connecting a supported assistant with your agreement. It also teaches the safe flow: write a profile, check it with `ripeat validate`, preview it with
`ripeat plan` and `ripeat assess --json`, show the risk in plain words, and **hand the person the `ripeat install` command to run themselves**. It forbids running the commands that change the machine,
adding `--yes`, `--accept-risk` or `--allow-admin`, inventing package names or recipes, and working around a HIGH or BLOCKED rating.

The [skills installer](https://github.com/vercel-labs/skills) can install it from the public documentation site. It requires Node.js and asks which agent and installation scope to use:

```bash
npx skills add https://ripeat.github.io/skills/ripeat/SKILL.md --skill ripeat
```

Alternatively, save [the file](https://ripeat.github.io/skills/ripeat/SKILL.md) as `~/.claude/skills/ripeat/SKILL.md` (Claude Code; other agent tools have their own skills folder):

```bash
mkdir -p ~/.claude/skills/ripeat
curl -fsSL https://ripeat.github.io/skills/ripeat/SKILL.md -o ~/.claude/skills/ripeat/SKILL.md
```

A skill is an instruction, not a lock: an agent with a shell can ignore it. The real protections are ripeat's own confirmations (`--yes` does not cover a HIGH-rated plan) and the
[administrator policy](./admin-policy). If your assistant supports MCP, [`ripeat mcp setup`](./mcp#connecting) is safer, because its tools cannot install at all unless you switch that on.

## `llms.txt`

[`https://ripeat.github.io/llms.txt`](https://ripeat.github.io/llms.txt) is a plain-text index of these docs for agents that read the web: a one-paragraph summary, then each page with its title and a link to its
Markdown source (for example `https://ripeat.github.io/commands.md`). It is built from the same list as the sidebar, so it never lists a page that is not published.
