# Finegrained Sync

This repository distributes public releases and documentation for Finegrained
Sync. It does not contain the sync tool's source code.

Finegrained Sync runs on an authorized user's computer. It can synchronize
Claude Code and OpenAI Codex activity to a Finegrained tenant, but only after
the user signs in and the tenant's collection policy authorizes that work.

The installed command-line tool is named `finegrained-sync`.

## Release availability

There are no downloadable releases yet.

| Platform | Current availability | Installation model |
| --- | --- | --- |
| macOS | First supported release in preparation | Signed or unsigned `.pkg` installer, depending on the release |
| Windows | Not supported for end-user installation yet | The project can build a CLI binary, but a Windows installer, background service, and secure credential storage are not ready |
| Linux | Not supported for end-user installation yet | The project can build a CLI binary, but a Linux package, background service, and secure credential storage are not ready |

When releases are published, download them only from this repository's
[Releases](https://github.com/finegrained-io/finegrained-sync-releases/releases)
page. The Finegrained web app uses stable links to the latest release; each
production release must publish the same asset names:

```text
finegrained-sync-macos.pkg
finegrained-sync-windows.msi
finegrained-sync-linux.tar.gz
SHA256SUMS
```

## Before installation

Your Finegrained administrator must configure the tenant's collection policy
before you enable synchronization. That policy controls whether Finegrained
Sync may discover sessions, synchronize trace segments, or synchronize matching
files.

You will need your tenant's HTTPS URL, for example:

```text
https://app.example.finegrained.io
```

Do not use a shared device token or put a device token in a shell command or
configuration file. The normal login flow opens your browser and creates a
scoped credential for this computer.

## macOS installation

These steps apply once the first macOS release is available.

1. Download `finegrained-sync-macos.pkg` from the release page.
2. Verify the download against the `SHA256SUMS` file attached to the same
   release:

   ```sh
   shasum -a 256 -c SHA256SUMS
   ```

3. Open the package and complete the installer. It installs
   `finegrained-sync` at `/usr/local/bin/finegrained-sync` and registers a
   per-user background sync process.
4. Confirm the installation and enroll the computer:

   ```sh
   finegrained-sync version
   finegrained-sync login -tenant https://YOUR-TENANT-URL
   finegrained-sync whoami
   finegrained-sync doctor
   ```

The login command opens a browser window for your Finegrained sign-in and
stores the resulting device credential in your macOS Keychain.

### Unsigned preview packages

Early packages may not be signed or notarized. macOS will warn before opening
an installer from an unidentified developer. Only continue when you downloaded
the package from this repository's release page and have confirmed its checksum
with your Finegrained contact. After the initial warning, open **System
Settings → Privacy & Security** and choose **Open Anyway** if your organization
has approved the installation.

## What Finegrained Sync can synchronize

Finegrained Sync can discover Claude Code sessions under `~/.claude/projects`
and read OpenAI Codex thread items from `~/.codex/thread_history_1.sqlite`.

- Nothing is synchronized before login and an active tenant policy permits it.
- Session discovery reports opaque session and project identifiers; it does not
  include an absolute local path or trace contents.
- Trace and file synchronization are limited by the tenant's policy and the
  local allow/deny configuration.
- Common credentials, including API keys, tokens, passwords, URL credentials,
  AWS access keys, and PEM private-key blocks, are redacted before upload.
- This is not a general personal-information or business-document redactor.
  Your organization's policy must authorize the directories and work data it
  chooses to synchronize.

## Everyday commands

```sh
# Inspect enrollment, policy cache, and local sources.
finegrained-sync status
finegrained-sync doctor

# Preview a single sync without uploading data.
finegrained-sync once --dry-run

# Pause or resume background synchronization without removing the tool.
finegrained-sync pause
finegrained-sync resume

# Inspect recent sync output.
finegrained-sync logs -n 100

# Remove Finegrained Sync and revoke its device credential.
finegrained-sync uninstall --dry-run
finegrained-sync uninstall
```

`uninstall` removes the background sync process, package receipt, CLI, local
configuration and state, and the Keychain credential. It may request
administrator approval.

## Troubleshooting

| Symptom | What to do |
| --- | --- |
| `command not found: finegrained-sync` | Run `/usr/local/bin/finegrained-sync version`. If it works, add `/usr/local/bin` to your shell `PATH`; otherwise reinstall the package. |
| Browser sign-in does not finish | Run `finegrained-sync login -tenant https://YOUR-TENANT-URL` again and complete the browser sign-in. The tenant URL must use HTTPS. |
| No data synchronizes after sign-in | Run `finegrained-sync status` and `finegrained-sync doctor`, then ask your administrator to confirm the tenant policy permits the intended synchronization. |
| You need help | Send your Finegrained contact the output of `version`, `status`, `doctor`, and `logs -n 100`. Do not include device tokens, raw traces, absolute paths, or unredacted file contents. |

## Security issues

Please read [SECURITY.md](SECURITY.md). Do not report vulnerabilities in a
public GitHub issue.
