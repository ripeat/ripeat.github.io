# Vendor installers: the `installer` method

For apps a vendor offers as a **disk image (`.dmg`, macOS)**, an **installer package (`.pkg`, macOS; `.msi`, Windows)** or a **setup program (`.exe`, Windows)**
and that no package manager carries. ripeat installs only what it can later undo; see "What it does" for how
each kind is undone.

```yaml
platforms:
  darwin:
    method: installer
    package: ollama
    url: https://example.com/Ollama.dmg
    sha256: 5ac60dd9787eef6539403d72740e9d153094fc9e998192025048bee812b86094
    app: Ollama.app          # the app inside the image
    signer: 'Developer ID Application: Ollama Inc. (ABCDE12345)'
    scope: user              # the app goes into ~/Applications, no admin rights
  windows:
    method: installer
    package: ollama
    url: https://example.com/Ollama.msi
    sha256: 0a1b...          # 64 hex characters
    signer: Ollama Inc.
```

| Key | Meaning |
|---|---|
| `url` | The download address. Plain `http` draws a warning |
| `sha256` | Required: the checksum the vendor publishes. The download must match it, or nothing is installed |
| `kind` | `dmg`, `pkg`, `msi` or `exe`; read from the end of the address when left out |
| `app` | dmg only, required: the app to copy out of the image (a plain name ending in `.app`) |
| `signer` | Text the vendor's signing name must contain. If it does not, the install is refused |
| `unsigned` | `true` accepts a file with no valid vendor signature. Rated HIGH; leave it out |
| `product` | Windows: part of the name the app shows in "Add or remove programs" (`Node.js`). It is the check: the tool counts as installed when a program with this in its name is listed. See below |
| `args` | `.exe` only, required: the plain flags that make the setup program install silently (`/S` for NSIS, `/VERYSILENT /NORESTART` for Inno Setup; the vendor's documentation says which). Each must look like `/S`, `-q`, `--silent` or `/D=C:\Tools\x`: no spaces, quotes or shell characters |
| `uninstall_args` | `.exe` only: the silent flags for its uninstaller, when ripeat does not know them (see below) |
| `scope` | `user` puts a dmg's app in `~/Applications` (no admin rights); otherwise `/Applications` |

## Two checks before anything runs

1. **The SHA-256** must match the recipe's. A mismatch stops the install.
2. **The vendor's signature** must be valid. A checksum copied from the vendor's own page only proves the file arrived intact; a
   signature proves who made it. On macOS ripeat asks `codesign` and `spctl` (Gatekeeper must accept the app, which means a Developer
   ID signature and, for current apps, notarization). On Windows it asks `Get-AuthenticodeSignature` for a `Valid` signature. The signer
   is shown during the install, and `signer:` pins who it must be. A file without a valid signature is refused unless the recipe says
   `unsigned: true`.

## What it does

- **dmg:** mounts the image read-only, copies that one `.app` into the Applications folder, and unmounts. Nothing inside the image is
  run. If an app of that name is already there, ripeat stops: it never replaces an app it did not install.
  **Undo** deletes exactly that app and nothing else.
- **msi:** runs `msiexec /i` quietly (`/qn /norestart`) with administrator rights. The file is kept in
  `$RIPEAT_HOME/installers/<package>/`, and **undo** runs `msiexec /x` on it, then deletes it. If the installer asks for a restart
  (exit code 3010 or 1641), ripeat reports it as done and says a restart is needed.
- **pkg:** a `.pkg` can write anywhere, so ripeat reads it first. It checks the SHA-256 and the signature (`pkgutil --check-signature`: a certificate macOS trusts, and `spctl` must
  accept the package, which for current packages means notarized), lists what it would install (`pkgutil --payload-files`), and **refuses** a package that:
  writes outside `/Applications`, `/usr/local`, `/opt` and `/Library/Application Support`; installs a launch daemon or agent, a privileged helper, a system extension or a kernel extension (anywhere in its paths);
  or would overwrite a file that already exists. It never replaces something it did not install. It then runs `installer -pkg … -target /` (administrator rights, asked once up front), and
  records the paths the package created (new paths whose parent already existed, so a shared folder like `/usr/local/bin` is never recorded) and the receipt ids it added, in `$RIPEAT_HOME/installers/<package>/pkg.json`.
  **Undo** re-checks every recorded path (it will not remove a system folder), removes them, and runs `pkgutil --forget` on the receipts. If the package has install scripts, the plan says so and undo is partial: ripeat removes what
  the package's files and receipt list and cannot undo what a script did. The draft's `check` is `pkgutil --pkg-info <receipt id>`, and `ripeat add` refuses a package that declares no receipt id.
- **exe:** an `.exe` writes wherever it likes, so ripeat undoes it only by what it registered in "Add or remove programs". It requires `product`, checks the
  SHA-256 and the signature, lists the programs, runs the file directly (no shell) with the recipe's `args`, and lists again. The one new entry whose name contains
  `product` is saved in `$RIPEAT_HOME/installers/<package>/uninstall.json`. If a program with that name is already listed, ripeat does not run the installer at all.
  If no single new entry appears, the install still counts, ripeat says it cannot undo it, and you remove it in Settings > Apps.
  **Undo** runs the entry's own uninstaller quietly (its `QuietUninstallString`; else `msiexec /x {code} /qn` for an MSI-based entry, `/VERYSILENT /NORESTART` for an Inno Setup
  `unins000.exe`, `/S` for an NSIS `uninstall.exe`; else the recipe's `uninstall_args`; an uninstaller ripeat does not recognise is not guessed). The uninstaller must be signed
  by the **same signer** as the installer, or ripeat does not run it. Afterwards ripeat checks the entry is gone. It does not undo data the app wrote itself, or what the installer did
  without an entry (services, file associations). A system install needs an elevated terminal, as for an `.msi`.

## Risk

A dmg is MEDIUM: it copies an app and runs nothing. A pkg, an msi or an exe is HIGH: it runs the vendor's installer code with administrator rights (an exe, or a pkg that runs install scripts, can undo only part of what it did),
so the plan needs your `--accept-risk` or a yes at the terminal, and that one answer also counts as the install confirmation
([Risk and `assess`](./assess)). `unsigned: true` makes either HIGH.
The administrator policy can deny the method (`deny_methods: [installer]`) or limit download hosts with `allow_domains`
([Administrator policy](./admin-policy)).

## Creating one: `ripeat add <name> --url ... --sha256 ...`

```bash
ripeat add ollama --url https://example.com/Ollama.dmg --sha256 5ac60dd9...
```

You supply the checksum from the vendor's page. ripeat never takes one from the file it downloads, because a checksum worked out from the
file vouches for nothing. It then: asks before downloading; downloads and checks the file against your checksum; reads who signed it and
(for a dmg) which app is inside; refuses an unsigned file unless you add `--unsigned`; pins the signer in the draft; and shows the recipe
for your approval. Flags: `--kind` (when the address does not end in `.dmg`, `.pkg`, `.msi` or `.exe`), `--app` (when the image holds several apps),
`--signer` (to pin a different part of the signer's name), `--product` (what the app is called in "Add or remove programs"; an msi: read from the package itself on Windows when you leave it out; an exe: required), `--args` (an exe: the silent flags, required), `--uninstall-args` (an exe: for its uninstaller), `--unsigned`.
A dmg draft is `scope: user`.

## Limits

- **Fixed address, fixed version.** A vendor that replaces the file at the same address breaks the checksum: ripeat refuses the new file
  until you update the recipe (`ripeat recipes check` only checks that the address answers, not the checksum).
- **Not from a recipe pack.** A pack may not carry an `.exe` or `.pkg` installer ([Recipe packs (taps)](./taps)), and an assistant never drafts or runs one.
- **An msi or exe is only recognised as installed through `product`.** On Windows ripeat reads "Add or remove programs" itself (the machine-wide list, in both its 64-bit and 32-bit views, and the current user's;
  entries Windows hides as system components are left out) and counts the tool as installed when a program whose name contains `product` is listed, ignoring case. That is a registry read, not a command, so it works the same
  in a recipe pack. Without a `product` (or a `check`), ripeat cannot tell the app is there, and `ripeat validate` warns.
- **The pkg flow is verified by unit tests only** (path rules, owned-path choice, signature and receipt parsing, refusing a damaged record). It has not been run against a real package, and no CI job builds one yet ([What has been verified](./platforms)).
- **The exe flow is verified by unit tests only** (registry diff, command-line parsing, uninstall choice, the signer rule, refusals, run against stand-in programs). It has not been run against a real
  vendor installer on Windows ([What has been verified](./platforms)).
- **Verified so far:** the **msi** flow on a real Windows 11 ARM64 machine with Node.js's official, OpenJS Foundation-signed installer: `ripeat add`
  checked the checksum and read the signer, `install` ran `msiexec`, `node --version` worked, and `rollback` ran `msiexec /x` and deleted the
  saved file. The **dmg** flow (mount, signature refusal, copy, undo, unmount) against a real disk image in a temporary folder on macOS, and the parsing
  of macOS and Windows signature output. A dmg from a real, signed and notarized vendor download is **not** yet verified
  ([What has been verified](./platforms)).
