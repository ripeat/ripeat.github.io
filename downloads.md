# Installs from GitHub releases and URLs

For tools that no package manager carries. Methods: `github_release` and `url`
([Recipes](./recipes)). Both install **without admin rights** and refuse to
install anything whose SHA-256 is unknown or does not match.

## `github_release`

```yaml
platforms:
  linux:
    method: github_release
    package: ripgrep                  # the app's folder name
    scope: user
    repo: BurntSushi/ripgrep
    asset: 'ripgrep-[0-9.]+-{arch_gnu}-unknown-linux-musl\.tar\.gz'
    checksum_asset: '{asset}.sha256'  # only needed when the release has no digest
    bins: [rg]                        # programs to put on PATH
```

- **Which release:** the latest, or the tag you pinned (`ripgrep@15.2.0`; ripeat
  tries `15.2.0`, then `v15.2.0`).
- **Which file:** `asset` is a regular expression matched against the whole file
  name. `{os}` becomes `linux`/`windows`, `{arch}` `amd64`/`arm64`, `{arch_gnu}`
  `x86_64`/`aarch64`. **Exactly one** asset must match; zero or several is an error,
  never a guess.
- **Which checksum:** GitHub's own SHA-256 for the asset if it has one; otherwise the
  file named by `checksum_asset` (`{asset}` is the chosen file). A file may hold one
  digest or `digest  name` lines. No checksum found means no install.
- **Rate limits:** set `GITHUB_TOKEN` to raise GitHub's API limit.

## `url`

```yaml
    method: url
    package: tool
    url: https://example.com/tool-{os}-{arch}.tar.gz
    sha256: 0123...64 hex characters
    bins: [tool]
```

`sha256` is required. A version cannot be pinned (ripeat warns).

## What happens on install

1. Download, verifying SHA-256 while it downloads. A mismatch installs nothing.
2. Unpack `.tar.gz` or `.zip` into a temporary folder. Entries that would escape
   the folder are an error; links inside the archive are skipped. A file that is
   not an archive is installed as the single program.
3. If the archive wraps everything in one folder, that folder becomes the root.
   Every name in `bins` must exist inside it.
4. Move it to `$RIPEAT_HOME/apps/<package>/` and link each program into
   `$RIPEAT_HOME/bin/` (a symlink; a copy on Windows).

ripeat prints a reminder to add `$RIPEAT_HOME/bin` to your `PATH`. It does not edit
your shell files. Its own probes (`check`, `version`) see that folder.

## Uninstall

Removes the links named in `bins` and the app folder, nothing else. Other files in
`bin/` are never touched.

## Limits

- Updates are not checked: `outdated` says "cannot check yet" for these tools.
- On macOS the downloaded program is not notarized by ripeat; files fetched with `curl`
  (as ripeat does) are not quarantined by Gatekeeper, but ones you download in a browser may be.
