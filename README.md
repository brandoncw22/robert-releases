# Robert

Robert is a local-first AI assistant for your terminal. It talks to models running on your own
computer (through [Ollama](https://ollama.com) or an OpenAI-compatible server you configure). It
needs no account, subscription, or internet connection once your models are available, and it sends
no telemetry.

This repository will publish Robert's Windows and Linux portable archives, script installers,
`exec-json.schema.json`, and `SHA256SUMS`. Application source is kept out of this repository.

**No release has been published yet.** When a version is tagged, the assets will appear on the
[Releases](https://github.com/brandoncw22/robert-releases/releases) page. Until then there is
nothing to download here.

## Supported platforms

- **Windows 11 x64** is the primary target. **Windows 10 x64** is supported but untested.
- **Linux x64**

macOS is not supported.

## Prerequisites

Install these on the machine where Robert will run. They are not included in the archive or the
script installers.

- **Node.js 22.13 or newer** on your `PATH`. Download it from [nodejs.org](https://nodejs.org/).
  Node is not bundled. A portable archive does not install itself or change `PATH`. The script
  installers do not bundle or download Node.js either.
- A model server: [Ollama](https://ollama.com/download) (the default), or an OpenAI-compatible
  server you configure. This repository does not require a specific model name. `robert doctor`
  reports what is missing once Robert is running.

## Release assets

Each GitHub Release (when one exists) has exactly these six assets:

- `robert-<version>-windows-x64.zip`
- `robert-<version>-linux-x64.tar.gz`
- `robert-<version>-windows-x64-install.ps1`
- `robert-<version>-linux-x64-install.mjs`
- `exec-json.schema.json` (the `robert exec --json` schema, version 1)
- `SHA256SUMS`

`SHA256SUMS` covers every other asset above, including both script installers and the schema. It
does not cover itself. Each line is a lowercase SHA-256 digest, two spaces, then the file name.

There is no `RobertSetup.exe`. Windows setup is `robert-<version>-windows-x64-install.ps1`. The
files are not Authenticode-signed, and no separate signature file (`.sig`, `.asc`) is published.

## Install

Replace `<version>` with the version on the release page (for example `0.1.0` or `0.1.0-rc.1`).

Portable archives do not install themselves or change `PATH`. Script installers are a separate
path, described below and in the source guides:
[Install Robert on Windows](https://github.com/brandoncw22/Robert/blob/main/docs/install-windows.md)
and
[Install Robert on Linux](https://github.com/brandoncw22/Robert/blob/main/docs/install-linux.md).

### Windows

1. Download `robert-<version>-windows-x64.zip` and `SHA256SUMS` from the
   [Releases](https://github.com/brandoncw22/robert-releases/releases) page. To check every
   checksum line, also download the other assets into the same empty directory: the Linux
   archive, both script installers, and `exec-json.schema.json`.
2. Verify the zip. PowerShell's `Get-FileHash` prints **uppercase** hashes; `SHA256SUMS` is
   **lowercase**. Compare them case-insensitively (`-eq` is case-insensitive for strings):

   ```powershell
   $archive = Get-Item .\robert-<version>-windows-x64.zip
   $actual = (Get-FileHash -Algorithm SHA256 -Path $archive).Hash
   $expected = (
     Get-Content .\SHA256SUMS |
       Where-Object { $_ -match 'robert-.*-windows-x64\.zip$' } |
       ForEach-Object { ($_ -split '\s+')[0] }
   )
   if ($actual -eq $expected) { 'OK' } else { 'MISMATCH' }
   ```

   Verify the Windows script installer as well. `SHA256SUMS` puts two spaces before the file
   name. `-ine` compares hashes without caring about letter case:

   ```powershell
   $file = 'robert-<version>-windows-x64-install.ps1'
   $line = @(Get-Content .\SHA256SUMS | Where-Object { $_.EndsWith("  $file") })
   if ($line.Count -ne 1) { throw 'Expected exactly one installer checksum' }
   $expected = ($line[0] -split '\s+')[0]
   if ((Get-FileHash -LiteralPath $file -Algorithm SHA256).Hash -ine $expected) { throw 'Checksum mismatch' }
   ```

   If either result is a mismatch, delete the download and fetch it again. Do not run a file
   whose hash does not match.
3. Extract the zip. It unpacks a folder named `robert-<version>-windows-x64` containing
   `robert.cmd`, `robert.mjs`, `README.txt`, and `LICENSE`:

   ```powershell
   Expand-Archive -Path .\robert-<version>-windows-x64.zip -DestinationPath . -Force
   cd .\robert-<version>-windows-x64
   .\robert.cmd --version
   .\robert.cmd doctor
   ```

The launcher does not change `PATH`. You can add that folder to your user `PATH` yourself if you
want to run `robert` from any directory.

### Linux

1. Download `robert-<version>-linux-x64.tar.gz` and `SHA256SUMS`. To run `sha256sum -c SHA256SUMS`
   without errors about a missing file, download all six release assets into an empty directory.
2. Verify:

   ```sh
   sha256sum -c SHA256SUMS
   ```

   A match prints `OK` beside each filename, including both script installers and
   `exec-json.schema.json` when those files are in the directory. If a name in `SHA256SUMS` is
   not in the directory, that line fails. To check only the files you downloaded, including the
   Linux installer `robert-<version>-linux-x64-install.mjs`:

   ```sh
   sha256sum -c SHA256SUMS --ignore-missing
   ```

   Require `OK` on the installer line, not only on the archive. A failed line means that file
   does not match; delete it and download it again.
3. Extract and run. The tarball unpacks a directory named `robert-<version>-linux-x64` containing
   `robert`, `robert.mjs`, `README.txt`, and `LICENSE`. The packager sets the `robert` launcher to
   mode `0755`, and a normal `tar -xzf` keeps that executable bit. If a later copy drops it, run
   `chmod +x ./robert`.

   ```sh
   tar -xzf robert-<version>-linux-x64.tar.gz
   cd robert-<version>-linux-x64
   ./robert --version
   ./robert doctor
   ```

The launcher does not change `PATH`. Use `./robert` so you do not pick up a different command.

## Script installers

A release includes these two scripts in addition to the portable archives. There is still no
`RobertSetup.exe`. Both scripts are unsigned. Both require Node.js 22.13 or newer already on
`PATH`. They do not bundle or download Node.js, and they do not download or select a model.
Verify the script against `SHA256SUMS` before you run it. A matching checksum shows the bytes
were not altered. It does not prove who published the file.

### Windows script installer

`robert-<version>-windows-x64-install.ps1` is the current-user PowerShell setup (Windows
PowerShell 5.1). It does not need administrator rights. `ExecutionPolicy Bypass` on the command
below applies only to that process. It does not change machine or user policy. Enterprise
controls can still block the unsigned script.

After the installer checksum above:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File ".\robert-<version>-windows-x64-install.ps1" install
```

The default directory is `%LOCALAPPDATA%\Programs\Robert`. The installer adds that install's
`bin` folder to the **user** `PATH` without `setx` and without truncating other entries. It
registers **Robert CLI** in current-user uninstall settings. Open a new terminal,
then run `robert --version`, `robert --help`, and `robert doctor`.

For any other directory, pass the same `-InstallDir` on every install and uninstall.

Upgrade by running the newer installer. Installing that same version again is supported.
Downgrades are refused, and so is a same-version installer whose payload differs. Only one
current-user installation is registered at a time. An existing portable or developer launcher
is not replaced. `Get-Command robert -All` shows which `robert` comes first on `PATH`.

Uninstall from the current-user Windows application list, or, for the default directory:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File "$env:LOCALAPPDATA\Programs\Robert\uninstall.ps1" uninstall
```

Uninstall removes only verified files the installer owns, its registry entry, and a `PATH`
entry it originally added. Settings (`%APPDATA%\Robert`), instructions, SQLite sessions
(`%LOCALAPPDATA%\Robert`), and unrelated `PATH` entries stay. An interrupted install is not
rolled back automatically. Read the error and see which files remain before you delete
anything.

Details: [Install Robert on Windows](https://github.com/brandoncw22/Robert/blob/main/docs/install-windows.md).

### Linux script installer

`robert-<version>-linux-x64-install.mjs` is the user-level installer for Linux x64. It does not
use sudo, does not edit shell profiles, and does not download models.

```sh
sha256sum -c SHA256SUMS --ignore-missing
node ./robert-<version>-linux-x64-install.mjs install
$HOME/.local/bin/robert --version
$HOME/.local/bin/robert doctor
```

Require `OK` for `robert-<version>-linux-x64-install.mjs`, not only for other files. By
default, `~/.local/bin/robert` points at a versioned launcher under
`~/.local/lib/robert-cli/versions/`. If `~/.local/bin` is not on your `PATH`, add it yourself
and open a new terminal. A custom prefix uses the same `--prefix` for install and uninstall,
including a path with spaces:

```sh
node ./robert-<version>-linux-x64-install.mjs install --prefix "$HOME/My Robert 安装"
"$HOME/My Robert 安装/bin/robert" --version
```

Upgrade by running the newer installer with the same prefix. Reinstalling identical bytes is
idempotent. Downgrades are refused, and so is replacing the same version with different
content. Older version directories remain until uninstall. A portable archive already in that
prefix is not adopted. Remove its launcher yourself, or choose another prefix.

Uninstall with the installer you saved (repeat `--prefix` when you used one):

```sh
node ./robert-<version>-linux-x64-install.mjs uninstall
```

That removes only verified installer-owned version files and its launcher. Config, personal
instructions, SQLite sessions, unrelated files, and shell profiles stay. An interrupted run
can leave `.install-lock` or `.stage-*` directories. Do not remove a lock while an installer
is still running. Publishing a version directory and switching the launcher are separate
renames. There is no whole-install crash recovery.

Details: [Install Robert on Linux](https://github.com/brandoncw22/Robert/blob/main/docs/install-linux.md).

## First run

`robert doctor` (and `robert doctor --json`) is a read-only report of your machine, Node.js, and
provider status. It always exits 0 today, even when it reports problems. It does not save a
model and it does not call one.

`robert init` chooses an installed model for the provider you already configured. It does not
download models, switch providers, enable tools, or change permission policy. It needs Node.js
22.13 or newer and a reachable model server. Ollama is the default. A reachable server still
works when that server's own CLI is not on `PATH`.

```sh
robert init
robert init --yes
robert init --yes --skip-test
```

Interactive `robert init` lists installed models, tool-capable ones first, with capability
labels. Enter a number or an exact name. Enter alone chooses the first. A chat-only model
requires an explicit Yes and otherwise returns to the list. `--yes` selects the first
tool-capable model and does not prompt. It refuses when the only installed models are
chat-only. A global `--model <name>` is honored, and `--yes` still refuses a chat-only model
named that way.

The chosen name is saved in your personal config. Other settings are left alone. Environment
variables and flags still override the saved model. Unless you pass `--skip-test`, init then
sends one tool-free request: `Reply with the single word: ok`. Any nonempty reply is enough.
That checks connectivity. It does not show that the model can code or call tools. The default
timeout is 30 seconds. `--skip-test` skips the request and reports that the response is
unverified. If the probe fails later, the saved model remains so you can fix the server and
run init again.

The first interactive `robert`, when no model is selected, uses this same setup before chat.
Chat that already has a model, and session resume, keep the model they already use. One-shot
commands and `robert exec` never open the picker. Without a model they point you at
`robert init`.

This page does not recommend a particular model. Use `robert models` or the init list to see
what you have and whether each model can call tools.

Failure, cancellation, and exit codes:
[First-run setup](https://github.com/brandoncw22/Robert/blob/main/docs/onboarding-usage.md).

## Upgrade

For a portable archive, download and verify the newer archive for your platform, extract it, and
run the new launcher's `--version`. Settings and saved sessions live outside the extracted
folder, so replacing that folder does not delete them.

For a script install, run the newer installer with the same `-InstallDir` or `--prefix`. See
[Script installers](#script-installers).

## Uninstall

A portable folder is not an installed program, so that copy has no uninstaller.

1. Delete the extracted `robert-<version>-windows-x64` or `robert-<version>-linux-x64` folder, and
   the downloaded archive if you no longer need it.
2. Optionally delete settings and sessions:
   - Windows: `%APPDATA%\Robert` and `%LOCALAPPDATA%\Robert`
   - Linux: `~/.config/robert` and `~/.local/share/robert` (or `$XDG_CONFIG_HOME` / `$XDG_DATA_HOME`)
   - If you set `ROBERT_HOME`, delete that folder instead
3. If you added the extract folder to `PATH`, remove that entry.

The session database file is `robert.db` in the data folder (plus SQLite WAL sidecars
`robert.db-wal` and `robert.db-shm` if they exist). Robert does not use a separate cache
directory.

To remove a script installation, use the uninstall command under
[Script installers](#script-installers). That uninstall leaves settings and sessions in place
unless you delete them yourself.

## Bugs

Report a problem with a released build by opening a
[bug report](https://github.com/brandoncw22/robert-releases/issues/new?template=bug_report.yml).

Include:

- Robert version
- Operating system (Windows or Linux)
- Provider and the model you used (for example Ollama and the model name)
- What you did, what you expected, and what happened
- The output of `robert doctor`

The report is public. Remove secrets first: tokens, passwords, API keys, cookies, and private
paths you do not want published.

## License and terms

This software is proprietary. All rights reserved. See [`LICENSE`](LICENSE).
