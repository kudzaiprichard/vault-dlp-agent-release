# Changelog

What changed in each release of the DLP Agent. Downloads are on the [releases page](https://github.com/kudzaiprichard/vault-dlp-agent-release/releases).

## v0.2.2 (2026-11-07)

### Added

- brand the block page
- show the Vault mark, wordmark and colours in the app
- use the Vault mark for the app, installer and tray icons

### Fixed

- don't inspect the signing fields of presigned upload forms
- drop internal changelog sections from release bodies with CRLF line endings

## v0.2.1 (2026-11-07)

### Fixed

- bump versions in place so the config files keep their formatting
- allow a session token in a token field sent to a site's own widget
- hide credential field values in the context of a snippet
- record a retried block once and name uploads the site announced
- answer a page's scripts with a short message instead of the block page

## v0.2.0 (2026-11-07)

### Added

- use settings and the server certificate from the installer's folder
- connect to the server from Settings with a connection test

### Fixed

- log why the server refused an incident batch before splitting it
- drop model findings that are timestamps or today's date
- retry an unreachable server at least once per heartbeat interval
- report an administrator's restart as agent_stopped reason restart
- fetch the whole policy bundle and exact data after a new registration
- replace multi-line settings whole when merging a settings file
- keep incidents the server refuses and retry them after an update
- send the app as a plain file name, as the server expects
- always tunnel Chrome sync
- don't treat 13-digit timestamps as Visa card numbers
- don't offer WebSocket compression the relay can't read

## v0.1.0 (2026-11-06)

First release.

### Added

- check a settings file without installing
- accept a settings file of any agent.toml values at install time
- refuse blocked apps and apps that refuse inspection
- let a gate refuse connections before they are tunnelled
- ask for policy schema 3 by default
- queue coverage on disk and drop only the events the server flags
- download and match exact data, report encrypted files and negotiate schema 3
- add exact data downloads and schema 3 device facts to the client
- recognise password-protected archives and documents
- add exact data matching with salted truncated hashes
- match policies by channel
- add policy schema 3 types for channels, exact data, device control and blocked apps
- show the unassigned and key required connection states
- record agent events and minute metric samples and sync them
- add keyless registration, server config and the sync endpoint to the client
- add general encrypted queues with age and size limits
- add machine-wide lockdown so nothing reaches the web around the agent
- lock browsers to the agent so proxy and VPN extensions can't bypass it
- ship the server's AI service catalog for schema 1 and pre-sync bundles
- mark WebSocket traffic and optionally route command-line AI tools through the agent
- inspect client-to-server WebSocket messages
- sniff bodies of unknown type for JSON, protobuf, forms and UTF-8 or UTF-16 text
- attribute requests to apps and AI services, handle unreadable content and count coverage
- request policy schema 2 and send coverage reports
- report client connections, their programs and uninspected traffic
- resolve AI services like the server and target policies at them
- add policy bundle schema 2 types for AI services
- log request metadata for diagnosis and silence proxy-library connection noise
- read protobuf, Connect and gRPC bodies and Google json+protobuf
- build an NSIS installer that configures, trusts the CA and autostarts the agent
- add proxy supervisor, identity-provider bypass and opt-in trust on first use
- show admin server connection and disabled-by-administrator state
- sync with the admin server and apply its state, commands and policies live
- add agent API client with certificate trust, device info and mock server
- add encrypted SQLite store for sync state and the incident queue
- align policy bundle and engine with the server policy schema
- show incidents on the activity page and policies on status
- enforce policies with incidents, notifications and tray pause
- add policy engine, risk scoring and masking
- add policy bundle and action severity order
- add models page and detection status
- add tiered inspection with ML budget and model lifecycle
- add GLiNER2-PII detector on ONNX Runtime
- add zero-shot ml_label rule kind
- add verified, resumable model manager
- run rule detection on extracted content
- add built-in and Gitleaks secrets rule packs
- add rule engine with validators, context and allowlists
- add JSON rule pack format
- inspect extracted content in the proxy
- extract text from bodies, uploads, documents and images
- show live interception status with start and stop
- start interception on launch with crash-safe proxy restore
- add DPAPI, trust store and system proxy integration
- add inspecting MITM proxy with bypass list and fail-safe
- restyle pages with Windows 11 settings cards
- add custom title bar and Windows 11 navigation pane
- add Windows 11 light and dark theme
- frameless window with Windows 11 Mica backdrop
- add agent layout with sidebar, status header and pages
- add Tauri shell with config, logging, tray and plugins
- add Cargo workspace crates for the agent engine

### Fixed

- read device states from newer servers as pending
- never inspect captcha services, which broke sign-in on claude.ai
- allow signed tokens sent to sign-in endpoints
- fetch a full bundle after switching to a newer policy schema
- match card numbers at the end of a sentence
- send schema 1 servers incidents without the schema 2 fields
- match card numbers only in their usual groupings, not inside longer ids
- prefer rules over model on overlap and fix ZW ID, ZW phone and model false positives

### Performance

- scan long text in overlapping 1,000-character windows to bound model memory
- vendor gliner2-rs with ONNX Runtime thread spinning turned off
