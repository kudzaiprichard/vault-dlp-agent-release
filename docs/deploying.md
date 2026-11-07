# Deploying the DLP Agent

For IT administrators installing the agent on company PCs. Download the installer, its `.sha256`
and `agent-settings.example.toml` from the
[releases page](https://github.com/kudzaiprichard/vault-dlp-agent-release/releases/latest), and
check the download as shown in the [README](../README.md#check-the-download).

## Before you start

On the admin server's dashboard:

1. Optionally create a **registration key** (Dashboard → Registration keys) and choose whether PCs
   using it are approved automatically or wait under **Pending approval**. The key is shown once.
   Without a key, PCs join under **No key** and you assign one later.
2. If the server doesn't have a publicly trusted certificate (a server on your network, or on the
   same PC), get its root certificate, `root.crt`. A Vault server keeps it in
   `C:\Vault\agent\root.crt`. It lets the agent check the server's certificate properly.

## Three ways to install

### 1. A ready-made folder (easiest for users)

The admin server's **Devices → Download agent** page builds this for you as a zip. To make one by
hand, put these files in one folder next to the installer:

| File | Contents |
| --- | --- |
| `agent-settings.toml` | At least `[server]` with `url = "https://<server>"`. Optionally `registration_key` and any other [setting](settings.md). Start from `agent-settings.example.toml` |
| `root.crt` (or `server-ca.crt`, `server-ca.pem`) | The server's root certificate, if it needs one |

Users extract the folder and double-click the installer: nothing to type. The setup log
(`%LOCALAPPDATA%\DLP Agent\setup.log`) lists which files were used.

If `agent-settings.toml` holds a registration key, share the folder only with your staff, or leave
the key out: PCs then join under **No key** and you assign one on the dashboard.

### 2. Installer options (scripts and deployment tools)

```text
DLP-Agent_0.2.1_x64-setup.exe /S /SERVER=https://vault.example.com /KEY=vrk_... /CACERT=\\fileserver\it\root.crt
```

| Option | Meaning |
| --- | --- |
| `/S` | Silent: no wizard; the agent starts in the tray when it's done |
| `/SERVER=` | The admin server's address, `https://` only |
| `/KEY=` | Registration key from the dashboard |
| `/CACERT=` | The server's root certificate in PEM (`root.crt`) |
| `/PIN=` | Instead of `/CACERT`: the SHA-256 (hex) of a certificate in the server's chain. Pin the root or intermediate, not the site certificate, which is renewed often |
| `/CONFIG=` | A settings file with any other [settings](settings.md), merged in and checked at install. Start from `agent-settings.example.toml` |

All options are optional, and they win over the files in the installer's folder. Put values with
spaces in quotes: `/CACERT="C:\deploy files\root.crt"`.

| Server | Command |
| --- | --- |
| On the same PC | `... /S /SERVER=https://localhost /CACERT=C:\Vault\agent\root.crt` |
| On your network | `... /S /SERVER=https://vault-server /CACERT=\\fileserver\it\root.crt` |
| Hosted with a public certificate | `... /S /SERVER=https://vault.example.com` |
| None yet | `... /S` (connect later) |
| The same custom settings everywhere | add `/CONFIG=\\fileserver\it\agent-settings.toml` |

The installer runs as the signed-in user and installs for that user only. Run it in the user's
session (for example a logon script), not as SYSTEM. Even a silent install shows Windows'
certificate prompt once, and the user must click **Yes**.

Check a settings file before deploying it:

```text
"%LOCALAPPDATA%\DLP Agent\dlp-agent.exe" --check-config agent-settings.toml
```

### 3. Connect from the app

Install with no options and connect afterwards in the agent: **Status → Connect**, or
**Settings → Server connection**. It has a **This PC** button for a server on the same PC, finds a
local Vault server's certificate by itself, and has **Test connection** and **Save and connect**.

## What the installer does

1. Copies the agent to `%LOCALAPPDATA%\DLP Agent\` and adds a Start menu shortcut (and, for silent
   installs, a desktop shortcut).
2. Writes the settings to `%APPDATA%\com.dlp.agent\agent.toml`, and copies the server certificate
   next to it as `server-ca.pem`.
3. Creates the agent's own root certificate, **DLP Agent Local Root CA**, and adds it to the user's
   trusted certificates after Windows asks the user. This is what lets the agent read HTTPS
   uploads.
4. Makes the agent start with Windows, in the tray.

## First start

- The agent points the Windows proxy setting at itself. The previous setting is saved and restored
  whenever the agent stops.
- It registers with the server. The PC appears under **Devices**, either active or under
  **Pending approval**, depending on the key.
- Until it's approved, or if it can't reach the server, it enforces built-in policies: block
  private keys, card numbers and secrets; warn on ID numbers and bank details.
- It downloads the detection model (about 630 MB, once, resumable). Until then the status is
  **Degraded** (rules only).
- After approval it applies your policies within a minute, without a restart.

## Upgrades

Run the newer installer the same way. No options are needed: settings, the device's identity on
the dashboard, policies and queued reports are kept.

## PCs without internet

The agent needs no internet once the model is installed: detection happens on the PC, and a local
server is reached over the network.

1. On a PC that has downloaded the model, copy the folder
   `%LOCALAPPDATA%\com.dlp.agent\models\gliner2-privacy-pii` to a USB stick.
2. On the offline PC, install with `auto_download = false` under `[models]` in the settings file,
   open the agent, go to **Models → Import from folder** and choose the copied folder.
3. The files are checked against SHA-256 checksums built into the agent before use. The status then
   changes from **Degraded** to **Inspecting**.

## Locking browsers to the agent

With `lock_browsers = true` (the default), the agent sets per-user browser policies so Chrome,
Edge, Brave and Firefox must use it: proxy and VPN extensions are blocked and QUIC is turned off.

For stricter control, run `dlp-agent.exe --lockdown-machine` as administrator. It adds machine-wide
browser policies, locks the proxy setting, and sets a firewall rule that lets only the agent and
Windows services reach the web. Add `--dry-run` first to see what would change, and
`--allow-program PATH` for programs that must keep direct access. `--unlock-machine` undoes it.

## Uninstalling

From **Settings → Apps → Installed apps**, or silently with
`"%LOCALAPPDATA%\DLP Agent\uninstall.exe" /S`. A silent uninstall keeps the agent's data, so a
reinstall picks up where it left off. The PC stays on the dashboard until you remove it there.
