# Finegrained Sync

This repository distributes public releases and documentation for Finegrained
Sync. It does not contain the sync tool's source code.

Finegrained Sync runs on an authorized user's computer. It can synchronize
Claude Code and OpenAI Codex activity to a Finegrained tenant, but only after
the user signs in and the tenant's collection policy authorizes that work.

The installed command-line tool is named finegrained-sync.

## Release availability

Finegrained Sync 0.4.0 is available for macOS and Windows. Download the
newest release only from this repository's
[Releases](https://github.com/finegrained-io/finegrained-sync-releases/releases)
page.

| Platform | Current availability | Installation model |
| --- | --- | --- |
| macOS | Available | Per-user graphical .dmg installer and LaunchAgents |
| Windows | Available | Per-user .msi installer and Scheduled Tasks |
| Linux | No current release asset | Not published with 0.4.0 |

The 0.4.0 release includes these stable asset names:

~~~text
finegrained-sync-macos.dmg
finegrained-sync-windows.msi
SHA256SUMS
~~~

## Current installer trust status

Version 0.4.0 is the first signed release. Its macOS installer is Developer ID
signed and notarized, with the notarization ticket stapled to the DMG. Its
Windows MSI and bundled executables are Authenticode signed. The 0.3.3
installers were unsigned prototypes. Download from the release page and
verify the matching SHA-256 checksum before installing.

Signed automatic-update metadata and platform archives for 0.4.0 are hosted
at https://finegrained-io.github.io/finegrained-sync-releases/. An installed
client activates an update only when its tenant approves that exact version.

Download the matching SHA256SUMS file from the same release and verify the
specific installer before opening it:

~~~sh
grep ' finegrained-sync-macos.dmg$' SHA256SUMS | shasum -a 256 -c -
~~~

In PowerShell on Windows, compare the displayed hash with the
finegrained-sync-windows.msi entry in SHA256SUMS:

~~~powershell
Get-FileHash .\finegrained-sync-windows.msi -Algorithm SHA256
~~~

## Before installation

Your Finegrained administrator must configure the tenant's collection policy
before you enable synchronization. That policy controls whether Finegrained
Sync may discover sessions, synchronize trace segments, or synchronize matching
files.

You will need your tenant's HTTPS URL, for example:

~~~text
https://app.example.finegrained.io
~~~

Do not use a shared device token or put a device token in a shell command or
configuration file. The normal login flow opens your browser and creates a
scoped credential for this computer.

## macOS installation

1. Download finegrained-sync-macos.dmg and SHA256SUMS from the release page,
   then verify the checksum as described above.
2. Open the disk image and double-click **Finegrained Sync Installer**.
   The installer works only for the signed-in macOS user and does not request
   administrator access.
   If macOS rejects the installer, confirm that you downloaded version 0.4.0
   from this repository and contact Finegrained with the warning text.
3. Enter your workspace's HTTPS URL and complete the browser sign-in when
   prompted. If you defer sign-in, open a new Terminal window and run:

   ~~~sh
   finegrained-sync login -tenant https://YOUR-TENANT-URL
   ~~~

4. Confirm the installation:

   ~~~sh
   finegrained-sync version
   finegrained-sync whoami
   finegrained-sync doctor
   ~~~

The installer places the CLI at
~/Library/Application Support/Finegrained/Sync/bin/finegrained-sync, adds that
directory to standard shell profiles, and registers per-user collector and
maintenance LaunchAgents.
The login flow opens a browser and stores the device credential in your macOS
Keychain.

## Windows installation

1. Download finegrained-sync-windows.msi and SHA256SUMS from the release page,
   then compare the SHA-256 hash as described above.
2. Confirm that Windows reports a valid signature for the MSI:

   ~~~powershell
   (Get-AuthenticodeSignature .\finegrained-sync-windows.msi).Status
   ~~~

   If the browser or Windows blocks it, contact Finegrained with the warning
   text after verifying the checksum; a signed release may still need time to
   establish download reputation.
3. Run the MSI. It installs only for the current Windows user, then offers to
   connect the machine. Enter the workspace's HTTPS URL and complete the
   browser sign-in.
4. If you postpone sign-in, open **Finegrained Sync Setup** from the Start
   menu. You can also sign in from PowerShell:

   ~~~powershell
   $fg = "$env:LOCALAPPDATA\Finegrained\Sync\bin\finegrained-sync.exe"
   & $fg login -tenant https://YOUR-TENANT-URL
   ~~~

5. Confirm the installation:

   ~~~powershell
   $fg = "$env:LOCALAPPDATA\Finegrained\Sync\bin\finegrained-sync.exe"
   & $fg version
   & $fg whoami
   & $fg status
   ~~~

The MSI installs the CLI at
%LOCALAPPDATA%\Finegrained\Sync\bin\finegrained-sync.exe, provides a
**Finegrained Sync Setup** Start-menu shortcut, and registers per-user
collector and maintenance Scheduled Tasks. Windows on ARM can run this x64
release through Windows emulation.

## What Finegrained Sync can synchronize

Finegrained Sync can discover Claude Code sessions under ~/.claude/projects
and read OpenAI Codex thread items from ~/.codex/thread_history_1.sqlite.

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

~~~sh
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
~~~

uninstall removes the background sync process, local configuration and state,
and the device credential for the current user. To remove the Windows MSI
registration and Start-menu shortcut as well, uninstall **Finegrained Sync**
from Windows Installed apps.

## Troubleshooting

| Symptom | What to do |
| --- | --- |
| command not found: finegrained-sync | On macOS, open a new Terminal window after installation. On Windows, invoke $env:LOCALAPPDATA\Finegrained\Sync\bin\finegrained-sync.exe from PowerShell. |
| Windows setup says Finegrained Sync could not be found | Reinstall the newest MSI, then run **Finegrained Sync Setup** from the Start menu. |
| Browser sign-in does not finish | Run finegrained-sync login -tenant https://YOUR-TENANT-URL again and complete the browser sign-in. The tenant URL must use HTTPS. |
| No data synchronizes after sign-in | Run finegrained-sync status and finegrained-sync doctor, then ask your administrator to confirm the tenant policy permits the intended synchronization. |
| You need help | Send your Finegrained contact the output of version, status, doctor, and logs -n 100. Do not include device tokens, raw traces, absolute paths, or unredacted file contents. |

## Security issues

Please read [SECURITY.md](SECURITY.md). Do not report vulnerabilities in a
public GitHub issue.
