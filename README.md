# Robert

Robert is a local command-line coding assistant. It uses [Ollama](https://ollama.com/) on your own machine for its models. This repository will publish Robert's Windows and Linux release builds and their SHA-256 checksum files. Application source is kept out of this repository.

## Supported platforms

- Windows
- Linux

macOS is not supported.

## Prerequisites

Install these on the machine where Robert will run:

- **Ollama.** Download it from [ollama.com/download](https://ollama.com/download) and run it locally. Have the model you want available in Ollama. A required model name has not been published; the first release will record one if Robert needs a specific model.
- **Node.js.** Download it from [nodejs.org](https://nodejs.org/) and make sure `node` is on your PATH. A minimum Node.js version has not been published yet. The first release notes will give one if the build requires it, and will say if a packaged build already includes Node.js.

## Releases

Windows and Linux artifacts are attached to [GitHub Releases](https://github.com/brandoncw22/robert-releases/releases) in this repository, together with SHA-256 checksum files.

No release has been published yet. The Windows installer (`RobertSetup.exe`) and the Linux installer are **coming with the first release**.

## Install

### Windows

Coming with the first release.

1. Open the [Releases](https://github.com/brandoncw22/robert-releases/releases) page and download `RobertSetup.exe` plus the SHA-256 checksum file from the same release.
2. Verify the checksum. See [Verify a download](#verify-a-download).
3. Run `RobertSetup.exe` and finish the installer prompts.
4. Open a new terminal. If `robert` is not found, follow the note in that release.

### Linux

Coming with the first release.

1. Open the [Releases](https://github.com/brandoncw22/robert-releases/releases) page and download the Linux installer plus the SHA-256 checksum file from the same release.
2. Verify the checksum. See [Verify a download](#verify-a-download).
3. Run the installer using the command in that release's notes. The installer filename and the exact command will be published with the first release.

## Upgrade

Coming with the first release. When a newer release is published:

### Windows

1. Download the newer `RobertSetup.exe` and its SHA-256 checksum file from the newer release.
2. Verify the checksum.
3. Run `RobertSetup.exe` again.

The first release notes will say if an older install has to be removed before the new installer will run.

### Linux

1. Download the newer Linux installer and its SHA-256 checksum file.
2. Verify the checksum.
3. Run the installer using the command in that release's notes.

## Uninstall

Coming with the first release. An uninstaller is not published today.

### Windows

When `RobertSetup.exe` is available, remove Robert with the uninstall steps in that release. Those notes describe the install layout.

### Linux

When the Linux installer is available, remove Robert with the uninstall steps in that release.

## Verify a download

Compare every artifact to the SHA-256 checksum file from the same GitHub Release before you run it. A matching digest means the file is the one attached to that release. If it does not match, delete the download and fetch it again from the release page.

`RobertSetup.exe` and the Linux installer are coming with the first release. The examples below use placeholder names.

### Windows (PowerShell)

From the directory that contains the download:

```powershell
Get-FileHash -Algorithm SHA256 -Path .\ARTIFACT_FILE
Get-Content .\CHECKSUM_FILE
```

`Get-FileHash` prints a `Hash` value. That value must match the hexadecimal digest in the checksum file. Letter case does not matter. Replace `ARTIFACT_FILE` with the downloaded file's name and `CHECKSUM_FILE` with the checksum filename from the same release.

### Linux (`sha256sum`)

From the directory that contains the artifact and its checksum file. `sha256sum -c` expects each line to be a digest, two spaces, then the filename:

```bash
sha256sum -c CHECKSUM_FILE
```

Replace `CHECKSUM_FILE` with the checksum filename from the release. A match prints `OK` beside the filename. `FAILED` means the download does not match the published checksum.

To print a digest and compare it yourself:

```bash
sha256sum ./ARTIFACT_FILE
```

Replace `ARTIFACT_FILE` with the Linux artifact you downloaded.

## Bugs

Report a problem with a released build by opening a [bug report](https://github.com/brandoncw22/robert-releases/issues/new?template=bug_report.yml).

Include:

- Robert version
- Operating system (Windows or Linux)
- Ollama version and the model you used
- What you did, what you expected, and what happened
- The output of `robert doctor`

The report is public. Remove secrets first: tokens, passwords, API keys, cookies, and private paths you do not want published.

## License and terms

A license and terms have not been chosen. [`LICENSE`](LICENSE) is a placeholder for Brandon to replace before the first release.
