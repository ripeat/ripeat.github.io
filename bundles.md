# Bundles: a profile plus its recipes

A [profile](./profiles) names tools only, so it is safe to share but is no help for a tool the other person has no recipe for. A **bundle** is one file that holds a profile **and the recipes you added** that it needs,
so a whole setup moves as a single file or address.

```bash
ripeat bundle export work work.bundle.yaml        # you: write the saved profile "work" and its recipes to a file (without a file it prints it)
ripeat bundle import work.bundle.yaml             # a colleague: from the file
ripeat bundle import https://example.com/work.bundle.yaml --sha256 <hex>   # or from an address, checking the digest you were given
```

## Exporting

- Includes the recipes in your `recipes/` folder that the profile uses, and the ones those need through `requires`.
- Leaves out built-in recipes (the other person has them) and recipes that came from a recipe pack ([Recipe packs (taps)](./taps)); it names them so you can tell the other person to add the pack.
- Refuses to overwrite a file without `--force`, and prints the file's SHA-256 so you can publish it next to the file.

## Importing

A bundle can hold commands (a recipe may run a shell command or download a program), so importing is treated like adding a recipe pack:

- It needs a person at a terminal. There is **no `--yes`**. The administrator policy's `deny_taps` forbids it.
- It shows the bundle's SHA-256, the tools the profile asks for, and every recipe with its install method and risk rating, rated as strictly as a recipe you wrote yourself. `--sha256 <hex>` stops the import if the file is not the one the sender published.
- It applies the checks a pack gets: the recipe must be valid; no shell command unless you pass `--allow-shell` (read them first); no `.exe` or `.pkg` installer; no unsigned installer; `check` and `version` must be plain probes
  (a lookup of a version or a package, not a pipeline). A recipe that fails is listed with the reason and left out.
- It never replaces something you have. A recipe with the name of a built-in recipe is refused. A recipe with the same name as one of yours is kept as yours (and said to be identical or different). A profile with the same name needs another name or `--force`.
- Then it asks once: add the recipes that passed and save the profile. **Nothing is installed.** Check it with `ripeat assess --profile <name>`, then `ripeat install --profile <name>`.

## The format

```yaml
ripeat_bundle: 1
name: work
profile:
  version: 1
  tools:
    - git
    - acme
recipes:
  - name: acme
    check: acme --version
    platforms:
      linux: {method: apt, package: acme}
```

`ripeat_bundle: 1` is required, unknown keys are errors, and a bundle is one YAML document of at most 512 KB. `profile` is the same as a `ripeat.yaml`; each entry of `recipes` is the same as a recipe file ([Recipes](./recipes)).
