# Robert

Robert is a local-first AI assistant for your terminal. It talks to models running on your own
computer (through [Ollama](https://ollama.com) or an OpenAI-compatible server you configure). It
needs no account, subscription, or internet connection once your models are available, and it sends
no telemetry.

This repository publishes Robert's Windows and Linux **portable archives** and their SHA-256
checksum file. Application source is kept out of this repository.

**No release has been published yet.** When a version is tagged, the assets will appear on the
[Releases](https://github.com/brandoncw22/robert-releases/releases) page. Until then there is
nothing to download here.

## Supported platforms

- **Windows 11 x64** is the primary target. **Windows 10 x64** is supported but untested.
- **Linux x64**

macOS is not supported.

## Prerequisites

Install these on the machine where Robert will run. They are not included in the archive.

- **Node.js 22.13 or newer** on your `PATH`. Download it from [nodejs.org](https://nodejs.org/).
  Node is not bundled, and the archive does not install itself or change `PATH`.
- A model server: [Ollama](https://ollama.com/download) (the default), or an OpenAI-compatible
  server you configure. This repository does not require a specific model name. `robert doctor`
  reports what is missing once Robert is running.

## Release assets

Each GitHub Release (when one exists) has exactly these three assets:

- `robert-<version>-windows-x64.zip`
- `robert-<version>-linux-x64.tar.gz`
- `SHA256SUMS`

There is no `RobertSetup.exe` and no Linux installer in the release. Those installers are not
shipped yet. The files are not Authenticode-signed, and no separate signature file (`.sig`, `.asc`)
is published.

`SHA256SUMS` lists both archives. Each line is a lowercase SHA-256 digest, two spaces, then the
file name.

## Install

Replace `<version>` with the version on the release page (for example `0.1.0` or `0.1.0-rc.1`).

### Windows

1. Download `robert-<version>-windows-x64.zip` and `SHA256SUMS` from the
   [Releases](https://github.com/brandoncw22/robert-releases/releases) page. To check both checksum
   lines at once, also download the Linux archive into the same empty directory.
2. Verify the zip. PowerShell's `Get-FileHash` prints **uppercase** hashes; `SHA256SUMS` is
   **lowercase**. Compare them case-insensitively (`-eq` is case-insensitive for strings):

   ```powershell
   $archive = Get-Item .\robert-*-windows-x64.zip
   $actual = (Get-FileHash -Algorithm SHA256 -Path $archive).Hash
   $expected = (
     Get-Content .\SHA256SUMS |
       Where-Object { $_ -match 'robert-.*-windows-x64\.zip$' } |
       ForEach-Object { ($_ -split '\s+')[0] }
   )
   if ($actual -eq $expected) { 'OK' } else { 'MISMATCH' }
   ```

   If the result is `MISMATCH`, delete the download and fetch it again. Do not run a file whose
   hash does not match.
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
   without errors about a missing file, download all three release assets into an empty directory.
2. Verify:

   ```sh
   sha256sum -c SHA256SUMS
   ```

   A match prints `OK` beside each filename. If you only downloaded the Linux archive, compare just
   that file's line instead; `sha256sum -c SHA256SUMS` will fail on the missing Windows zip.
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

## First run

`robert doctor` (and `robert doctor --json`) is a read-only report of your machine, Node.js, and
provider status. It always exits 0 today, even when it reports problems.

**Today:** the first time you run `robert` in a terminal with no model set, it lists installed
models (tool-capable ones first) and saves your pick. One-shot commands and `robert exec` do not
show that picker; pass `--model <name>` or run interactive `robert` once.

A dedicated `robert init` setup command is coming (it is not in the current product). When it
ships, use `robert init` — or the first interactive `robert` with no model — to finish setup.

This page does not recommend a particular model. Use `robert models` or the first-run list to see
what you have and whether each model can call tools.

## Upgrade

Download and verify the newer archive for your platform, extract it, and run the new launcher's
`--version`. Settings and saved sessions live outside the extracted folder, so replacing that
folder does not delete them.

## Uninstall

There is no uninstaller, because this is a portable folder, not an installed program.

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
