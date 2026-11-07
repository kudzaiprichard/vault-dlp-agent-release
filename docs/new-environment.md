# Rolling out Vault in your organisation

How to set up the whole system from nothing: the Vault Server on one machine, and the DLP Agent on
every employee's PC. Read this once before you start; it takes about an hour, most of it the server
install. This guide covers the agent in detail; the server's own guide,
[Rolling out Vault (server)](https://github.com/kudzaiprichard/vault-dlp-server-release/blob/main/docs/rollout.md),
covers installing and running the server.

## What you're setting up

| Part | Where it runs | What it does | How often you set it up |
| --- | --- | --- | --- |
| **Vault Server** | One Windows machine that stays on (a server, or a PC that isn't switched off) | The dashboard where administrators set policies and see incidents. Agents get their policies from it and report to it | Once per organisation |
| **DLP Agent** | Every employee's Windows PC | Checks uploads and messages on the PC and blocks or reports sensitive data | Once per PC (a double-click) |

The agent protects a PC from the moment it is installed, even before it reaches the server: it
already blocks card numbers, private keys and secrets with its built-in policies. The server adds
your organisation's own policies, the dashboard and incident reports.

**The order is always:** install the server → sign in → upload the agent installer → download the
agent package → install that package on each PC → approve the PCs on the dashboard.

| Step | Who | How often |
| --- | --- | --- |
| [1. Plan](#1-plan) | Administrator | Once |
| [2. Install the server](#2-install-the-server) | Administrator | Once |
| [3. First sign-in](#3-first-sign-in) | Administrator | Once |
| [4. Upload the agent installer](#4-upload-the-agent-installer) | Administrator | Once per agent version |
| [5. Download the agent package](#5-download-the-agent-package) | Administrator | Once (and again for a new agent version or server address) |
| [6. Share the package](#6-share-the-package) | Administrator | Once |
| [7. Install the agent on each PC](#7-install-the-agent-on-each-pc) | Each employee, or IT | Once per PC |
| [8. Approve the PCs](#8-approve-the-pcs) | Administrator | Once per PC |
| [9. Check that it works](#9-check-that-it-works) | Administrator | Once |

## 1. Plan

Decide before you install:

- **Which machine runs the server.** Windows 10, 11 or Server 2019 or newer, switched on whenever
  employees work, with administrator rights for the install. The
  [Vault Server releases](https://github.com/kudzaiprichard/vault-dlp-server-release) page lists
  its full requirements.
- **The address employees' PCs will use to reach it.** A name such as `vault.company.local`, or a
  fixed IP address such as `192.168.1.10`. Give the server machine a fixed IP (or a DHCP
  reservation) so the address doesn't change. `localhost` only works on the server machine itself.
- **The network path.** Employees' PCs must reach the server on **port 443 (HTTPS)**. If the
  server machine's firewall or your network blocks it, allow it.
- **The employees' PCs.** Windows 10 or 11, 64-bit, about 1.5 GB of free memory and 700 MB of disk.
  Each PC downloads the detection model (about 630 MB) once on first start. For PCs without
  internet, see [PCs without internet](deploying.md#pcs-without-internet).

## 2. Install the server

Download the latest Vault Server from its
[releases page](https://github.com/kudzaiprichard/vault-dlp-server-release/releases/latest) and
follow the server's
[rollout guide](https://github.com/kudzaiprichard/vault-dlp-server-release/blob/main/docs/rollout.md).

When setup asks for the addresses the server answers on, include **every name and IP the
employees' PCs will use** (step 1). The server's HTTPS certificate covers only those addresses; an
agent that connects under any other name is refused. You can add addresses later with the server's
configuration tool (see the server's guide).

## 3. First sign-in

1. On the server machine, open `https://localhost` (or `https://<server>` from another PC) and
   sign in with the default administrator account: `admin@vault.local`, password `@Vault12`.
   Every Vault install starts with this same account.
2. **Change the administrator password straight away** (account menu → **Change your password**).
   While a default password is in use, the dashboard shows a red warning: anyone who can reach the
   dashboard could sign in with it. Better still, add your own administrator under **Users** and
   deactivate the default account.
3. Optionally, review **Policies** (card numbers, private keys and secrets are blocked from the
   start; the rest only log until you switch them on), and add **Users** and **Departments** with
   the employees' Windows usernames so incidents show who was involved.

## 4. Upload the agent installer

The server hands out the agent, so it needs a copy of the installer. You do this **once per agent
version**, not once per PC.

1. Open the agent's [latest release](https://github.com/kudzaiprichard/vault-dlp-agent-release/releases/latest)
   and download `DLP-Agent_<version>_x64-setup.exe` and `DLP-Agent_<version>_x64-setup.exe.sha256`.
2. On the dashboard, go to **Devices → Download agent**.
3. Under **Agent installer (.exe)**, choose the downloaded `.exe` (not the `.sha256`).
4. Open the `.sha256` file in Notepad, copy its contents, and paste them into
   **SHA-256 from the release page**.
5. Click **Upload installer**. The page shows the version, size and checksum.

The server refuses a file whose checksum doesn't match, so a damaged or swapped installer is never
handed out. The upload is recorded in the **Audit log**.

## 5. Download the agent package

On the same page, under **Agent package**:

1. **Server address the agents use:** choose the name or IP the employees' PCs reach the server on
   (step 1), **not** `localhost`. Only the addresses given during the server install are listed.
2. **Registration key:**
   - **No key (recommended):** each new PC appears on the dashboard as waiting, and you let it in
     (step 8). The package holds no secret, so it can be shared freely.
   - **Include a new key:** a key that lets PCs in straight away. Anyone with the zip can then join
     a PC, so share it only with your staff, and disable the key under **Registration keys** when
     you're done.
3. Click **Download agent package**. You get `DLP-Agent-setup-<address>.zip`, containing:

| File | What it is |
| --- | --- |
| `DLP-Agent_<version>_x64-setup.exe` | The agent installer you uploaded |
| `agent-settings.toml` | The server's address (and the key, if you chose one) |
| `root.crt` | The server's HTTPS certificate, so the agent can trust it |
| `README.txt` | Short instructions for the employee |

Keep this zip: the same one works for every PC. Download a new one only after uploading a new agent
version or changing the server's address.

## 6. Share the package

Put the zip where employees (or your IT staff) can get it: a shared folder, the intranet, email or
a USB stick.

### Silent and managed installs

To install on many PCs without anyone clicking through setup (Intune, Group Policy, PDQ, a
script), extract the zip and run the installer with `/S`. It still picks up `agent-settings.toml`
and `root.crt` from the folder it is in, so this is all it takes:

```text
DLP-Agent_<version>_x64-setup.exe /S
```

Run it from the extracted folder (or set that folder as the working directory). The install log
`%LOCALAPPDATA%\DLP Agent\setup.log` lists the settings files it used. Instead of the folder, you
can pass everything on the command line: `/SERVER=https://<server>`, `/CACERT=<root.crt>`,
`/KEY=<key>` and `/CONFIG=<settings file>` for any other setting. See
[Installer options](deploying.md#2-installer-options-scripts-and-deployment-tools).

Two things to know:

- **It installs for the signed-in Windows user**, so run it in the user's session (in Intune,
  install in the **user** context; with Group Policy, as a user logon script), not as SYSTEM.
- **The certificate prompt still appears**, even with `/S`: Windows always asks the user before
  a certificate is trusted for them. The user clicks **Yes** once. If nobody answers, the agent is
  installed but can't check secure uploads until it is accepted: the agent asks again the next
  time it starts.

## 7. Install the agent on each PC

On each PC, the employee (or IT):

1. **Extracts the zip:** right-click → **Extract All**. Running the setup from inside the zip
   leaves its settings behind and the agent can't find the server.
2. **Double-clicks `DLP-Agent_<version>_x64-setup.exe`** in the extracted folder. If Windows says
   **"Windows protected your PC"**, click **More info → Run anyway** (the installer isn't
   code-signed yet).
3. **Clicks Yes** when Windows asks about a certificate called **DLP Agent Local Root CA**. The
   agent needs it to check secure (HTTPS) uploads; it is created on the PC and never leaves it.
   The prompt can open behind other windows: if setup seems to stop, look for it on the taskbar.
   If it's missed, the agent asks again the next time it starts.

The agent starts in the system tray (the Vault icon by the clock) and starts with Windows from then
on. It installs for the current Windows user and needs no administrator rights. Until the detection
model has downloaded, the tray icon has an amber badge: protection is on, with rules only.

## 8. Approve the PCs

With **No key**, each PC appears under **Devices** as waiting. Open it and assign a registration
key to let it in (or create a key under **Registration keys** that approves PCs automatically). The
agent then gets your policies within a minute or two and shows **Connected** on its Status page.

## 9. Check that it works

On one PC:

1. The tray icon's tooltip says **DLP Agent: Inspecting traffic**.
2. Open ChatGPT (or another AI chat site) in the browser and send a message containing the test
   card number `4111 1111 1111 1111`. It's a well-known test number, not a real card. The message
   is blocked and an **Upload blocked** pop-up appears.
3. On the dashboard, the PC shows as active with its agent version, and the block appears under
   **Incidents**, masked (`4111-****-****-1111`).

## Later

| When | What to do |
| --- | --- |
| **A new agent version is released** | Upload it (step 4), then download a new package (step 5) for new PCs. For existing PCs, run the new installer on them (or the [silent upgrade](deploying.md#upgrades)): settings, the PC's identity and anything waiting to be sent are kept |
| **A new employee or PC** | Install the same package on it (step 7) and approve it (step 8) |
| **The server's address changes** | Add the new address with the server's configuration tool, download a new package, and reinstall it on the PCs, or change the address in each agent (Settings → Server connection) |
| **A PC is retired** | Uninstall the agent on it (**Settings → Apps → Installed apps → DLP Agent → Uninstall**, or `"%LOCALAPPDATA%\DLP Agent\uninstall.exe" /S`), then remove the PC under **Devices**. Uninstalling restores the proxy setting and removes the agent's certificate; see [Uninstall](../README.md#uninstall) and [Uninstalling](deploying.md#uninstalling) |

## When something goes wrong

| Problem | Check |
| --- | --- |
| The PC never appears under Devices | The address in the package is one the PC can reach (not `localhost`); port 443 is open; the zip was extracted before running setup. The install log `%LOCALAPPDATA%\DLP Agent\setup.log` shows which settings were used |
| The agent says the server's certificate isn't trusted | The package's `root.crt` was left out, or the PC uses an address that wasn't given during the server install |
| Uploads aren't checked | The certificate prompt was declined or missed: restart the agent (tray icon → Quit, then start **DLP Agent** from the Start menu) and click **Yes** |
| The tray icon keeps its amber badge | The detection model is still downloading, or the PC has no internet: see [PCs without internet](deploying.md#pcs-without-internet) |

More in [troubleshooting.md](troubleshooting.md), and in the server's own guide for the dashboard.
