# Troubleshooting

Where to look first:

- **Status page** (click the tray icon): protection state, certificate, system proxy, model,
  latency, admin server connection and policies, with the error when something failed.
- **Logs:** `%LOCALAPPDATA%\com.dlp.agent\logs\dlp-agent.YYYY-MM-DD.log` (values are masked). From
  the dashboard, **Send diagnostics** collects the last 2,000 lines without touching the PC.
- **Install log:** `%LOCALAPPDATA%\DLP Agent\setup.log`.

## Network and browsing

| Problem | Cause and fix |
| --- | --- |
| No internet after the agent crashed, was killed or the PC lost power | The Windows proxy setting still points at the stopped agent. Starting the agent restores it. Without starting it: `"%LOCALAPPDATA%\DLP Agent\dlp-agent.exe" --restore-proxy` |
| One site or app stops working | It probably pins its certificate or uses a company-internal certificate. Add its domain to `bypass_domains` in `agent.toml` ([settings.md](settings.md#proxy-traffic-interception)) and restart the agent (that site is then not inspected) |
| Microsoft / Google sign-in fails | Identity-provider sign-in (Microsoft, Google, Apple) is always passed through uninspected. If it still fails, check the log for the sign-in host and tell your administrator |
| Firefox shows certificate errors | Firefox has its own certificate store: set `security.enterprise_roots.enabled` to `true` in `about:config` |
| `curl.exe` fails with `CRYPT_E_NO_REVOCATION_CHECK` through the agent | Windows' curl insists on revocation data that the agent's per-site certificates don't carry; use `curl --ssl-no-revoke`. Browsers aren't affected |
| Uploads feel slow | Large documents are scanned up to the 400 ms budget and the rest in the background. Check the latency on the Status page; on slow PCs reduce `ml_groups` or set `ml_enabled = false` |

## Protection won't start

| Status page says | Fix |
| --- | --- |
| "Traffic inspection could not start: ... root certificate was not added" | The certificate prompt was declined. Click **Start protection** to be asked again, and choose Yes |
| "could not listen on 127.0.0.1:8899" | Another program uses the port. Change `proxy.listen_port` and restart |
| "Disabled by administrator" | Protection was turned off from the dashboard; only an admin can turn it back on |
| **Degraded** | The model is still downloading, failed its checksum, or couldn't load: see the **Models** page (retry the download, or Import from folder) |

## Admin server connection

The **Admin server** card on the Status page:

| Shows | Meaning and fix |
| --- | --- |
| **Connected** | All fine |
| **Waiting for approval** | Approve the device under Devices → Pending approval |
| **Offline** | The server can't be reached, or its certificate isn't trusted (hover for the error). Protection keeps running on the last policies, and incidents, events, coverage and metrics queue until it is back. Check the URL, that `https://<server>/api/health` opens on the PC, and the certificate settings |
| **Waiting for administrator** | The PC registered without a key. Assign a key to it on the dashboard (under *No key*); it takes effect at its next heartbeat |
| **Registration key required** | `agent.toml` has no key and the server doesn't accept PCs without one. Set `registration_key` and restart |
| **Registration key rejected** | The key in `agent.toml` is wrong, disabled, expired or used up. Fix `registration_key` and restart |
| **Device blocked by administrator** | Blocked on the dashboard; unblock it there (takes effect within the hour or at the next agent restart) |
| **Not configured** | No server `url` in `agent.toml` |

**Certificate errors to the server** ("invalid peer certificate", "does not contain the pinned
certificate"): set `ca_cert` to the server's CA (`C:\Vault\agent\root.crt` for the local server).
If a pin from first use is stale, delete `agent.db` and `agent.key` in `%APPDATA%\com.dlp.agent`
with the agent stopped: it registers again as the same device.

## Installer

| Problem | Fix |
| --- | --- |
| "Windows protected your PC" | The installer isn't code-signed: More info → Run anyway |
| `setup.log` shows a non-zero exit code | Read the message under it: for example "the server URL must use https", or a `/CACERT` file that isn't a PEM certificate. Fix the option and run the installer again |
| The agent doesn't start with Windows | Check Task Manager → Startup apps → DLP Agent is enabled; running the installer again re-registers it |
| A certificate removal prompt appears on uninstall | Expected: Windows asks before deleting the agent's root certificate. Click Yes |

## Resetting an agent

With the agent stopped (tray → Quit):

- **Settings only:** delete `%APPDATA%\com.dlp.agent\agent.toml`; the default is written on the
  next start (re-run the installer to fill in the server settings).
- **Server registration:** delete `agent.db` and `agent.key` in the same folder. The PC registers
  again as the same device; queued incidents that weren't uploaded are lost.
- **Everything:** uninstall with "delete the application data" ticked, then install again.
- **Only the root certificate**: in PowerShell,
  `Get-ChildItem Cert:\CurrentUser\Root | Where-Object Subject -like "*DLP Agent Local Root CA*" | Remove-Item`.
  The agent asks to trust a new one the next time protection starts.
