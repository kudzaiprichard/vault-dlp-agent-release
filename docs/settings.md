# Agent settings

The agent reads `%APPDATA%\com.dlp.agent\agent.toml`. It is created on first start (or by the
installer, with the server settings filled in), and every key in it is commented.

The same keys go in a settings file for the installer (`agent-settings.toml` next to it, or
`/CONFIG=`; see [deploying.md](deploying.md)). Start from `agent-settings.example.toml` on the
releases page.

- Edit the file, then restart the agent (tray → **Quit**, then start **DLP Agent** from the Start
  menu). Changes are read at start-up only.
- Every key is optional: anything left out uses the default below.
- Unknown keys are rejected, so typos are caught. If the file can't be read, the agent runs on the
  defaults and the Status page shows the error.
- The **policies themselves** (what is blocked, warned or logged) are not set here: they come from
  the admin server, or from the built-in defaults until the first sync.

## `[server]`: the admin server

| Key | Default | Meaning |
| --- | --- | --- |
| `url` | `"https://localhost"` | Server address, HTTPS only |
| `registration_key` | `""` | Organisation key from Dashboard → Registration keys. Empty = register **without a key**: the PC shows up on the dashboard under *No key* and waits until an admin assigns one (unless the server turns keyless registration off; then the Status page says a key is required) |
| `heartbeat_interval_secs` | `60` | Seconds between heartbeats until the server sends its own value |
| `policy_poll_interval_secs` | `60` | Seconds between policy checks (also checked whenever the heartbeat reports a new version) |
| `ca_cert` | `""` | The server's CA certificate in PEM. A relative path is relative to this folder; the installer copies `/CACERT=` here as `server-ca.pem` |
| `cert_sha256` | `""` | SHA-256 (hex) of a certificate in the server's chain, as an alternative to `ca_cert` |
| `trust_first_certificate` | `false` | With neither of the above, pin the certificate chain seen at first registration |
| `policy_schema` | `4` | Policy bundle schema to ask for. 4 adds risk levels (approval classes, behaviour baselines, level responses) to 3 (channels, exact data, blocked apps, device controls), which adds to 2 (AI services, app matching, unreadable content, coverage reports). If the server doesn't serve the newer one the agent steps down to 3 and then 2 by itself; set `1` only for much older servers |

### Settings that come from the server

Once registered, the server sends per-key settings with every heartbeat; they replace the values
above, are kept on the PC and stay in force offline:

| Setting | Range | Default before the server sends one |
| --- | --- | --- |
| Heartbeat interval | 10 s to 1 h | `heartbeat_interval_secs` |
| Sync interval (events, metrics) | 30 s to 1 h | 60 s |
| Offline notice after | 1 to 90 days | 7 days |
| Keep queued data for | 1 to 90 days | 30 days |
| Queue size limit (per queue) | 10 to 2048 MB | 200 MB |

### Server certificate

The agent only connects to the server over HTTPS and never through the system proxy. How it
checks the certificate:

1. **A publicly trusted certificate** (a real domain with a public CA): nothing to set.
2. **`ca_cert`** (recommended for the local server): the server's own CA, e.g.
   `C:\Vault\agent\root.crt` from the Vault server. Full chain and host name checks; survives the
   server's certificate renewals (Caddy renews its site certificate every 12 hours).
3. **`cert_sha256`**: a pin. Pin the CA or intermediate, not the site certificate, which rotates.
4. **`trust_first_certificate = true`**: trust whatever chain the server presents the first time.
   Only for servers whose certificates never rotate: with Caddy's internal CA the pinned
   intermediate changes about weekly and the agent then reports the server as offline.

Without any of these, a server with a self-signed certificate is refused and the Status page shows
it offline with the certificate error.

## `[proxy]`: traffic interception

| Key | Default | Meaning |
| --- | --- | --- |
| `listen_port` | `8899` | Local proxy port (listens on 127.0.0.1 only) |
| `fail_mode` | `"fail_open"` | If inspection crashes or times out: `fail_open` lets the request through, `fail_closed` blocks it |
| `scan_timeout_ms` | `3000` | Longest inspection of one request before the fail mode applies |
| `start_on_launch` | `true` | Start inspecting as soon as the agent starts |
| `set_system_proxy` | `true` | Point the Windows proxy setting at the agent while it runs (saved and restored) |
| `install_ca` | `true` | Make sure the agent's root certificate is trusted when protection starts |
| `bypass_domains` | Windows Update, Defender, SmartScreen, telemetry, Teams, Skype, Office measurement | Domains tunnelled without inspection; `example.com` covers its subdomains. Identity-provider sign-in endpoints are always added |
| `test_block_marker` | `"DLP-AGENT-TEST-BLOCK"` | Any upload containing this text is blocked, to test blocking end to end. `""` turns it off |
| `lock_browsers` | `true` | Lockdown level 1: browser policies so Chrome, Edge, Brave and Firefox must use the agent (proxy and VPN extensions blocked, QUIC off). See [deploying.md](deploying.md#locking-browsers-to-the-agent) |
| `cli_tools` | `false` | Route command-line AI tools (Claude Code, Codex, Node and Python CLIs) through the agent with user environment variables while protection runs. Leave off on developer PCs whose tools must not be inspected |

## `[limits]`: how much is inspected

| Key | Default | Meaning |
| --- | --- | --- |
| `max_body_bytes` | `10485760` (10 MiB) | Larger request bodies aren't inspected (logged) |
| `max_file_bytes` | `52428800` (50 MiB) | Larger files inside a request aren't extracted (logged) |
| `max_pdf_pages` | `200` | Pages read per PDF |
| `max_ocr_images_per_request` | `10` | Images run through OCR per request |
| `max_model_tokens_per_request` | `2048` | Text scanned by the model before the request is let through; the rest is scanned in the background |

## `[detection]`: rules and the ML model

| Key | Default | Meaning |
| --- | --- | --- |
| `ml_enabled` | `true` | Run the GLiNER2-PII model as well as the rules. `false` saves about 1 GB of RAM (rules only) |
| `ml_threads` | `4` | CPU threads for the model |
| `ml_groups` | `["person", "contact"]` | Built-in label groups: `person`, `contact`, `government_id`, `banking`, `digital_identity`, `secrets`, `sensitive_dates`. Each adds about 60 to 130 ms per request; the structured types are already found by rules |
| `ml_min_score` | `0.5` | Minimum model score for a finding |
| `ml_budget_ms` | `400` | Longest the model may delay a request |
| `large_content` | `"monitor"` | When the budget runs out: `monitor` lets the request through and reports late findings; `hold` waits for the full scan (up to `scan_timeout_ms`) |
| `hard_block_types` | `["private_key", "card_number"]` | Rule findings of these types skip the model (the request is blocked anyway). Enforced server policies marked "hard block" add their types |

## `[models]`

| Key | Default | Meaning |
| --- | --- | --- |
| `auto_download` | `true` | Download a missing model on start-up (about 630 MB, once, resumable). With `false`, use **Models → Download** or **Import from folder** |

## `[logging]`

| Key | Default | Meaning |
| --- | --- | --- |
| `level` | `"info"` | `trace`, `debug`, `info`, `warn`, `error`, or a filter such as `"info,dlp_proxy=debug"`. The `RUST_LOG` environment variable overrides it |
| `max_files` | `14` | Daily log files kept in `%LOCALAPPDATA%\com.dlp.agent\logs` |

## Command-line options

| Command | Purpose |
| --- | --- |
| `dlp-agent.exe` | Start the agent (window and tray) |
| `dlp-agent.exe --background` | Start in the tray without opening the window (used by autostart) |
| `dlp-agent.exe --restore-proxy` | Put the Windows proxy setting back without starting the agent |
| `dlp-agent.exe --check-config FILE` | Check a settings file before deploying it |
| `dlp-agent.exe --lockdown-machine [--dry-run] [--allow-program PATH]...` | Lockdown level 2 (run as administrator): machine-wide browser policies, locked proxy setting and a firewall that only lets the agent and Windows services reach the web. `--dry-run` prints what would change. See [deploying.md](deploying.md#locking-browsers-to-the-agent) |
| `dlp-agent.exe --unlock-machine` | Undo lockdown level 2 |
