**English** · [Français](README.fr.md)

# Relay for iPhone

Native iPhone client (SwiftUI) for a **Relay** server, a self-hosted connection manager
(SSH, RDP, VNC, credential vault, inventory sync). Version **0.4.0**.
Targets iOS 26.0+, French-language interface, no third-party dependencies.

## Features

- **QR code pairing**: Relay web → Administration → Devices. The device token (`rlt_…`) is stored
  in the Keychain, on this iPhone only, and can be revoked from Relay web.
- **Connections**: folder tree, global search, favorites, recents, availability indicator.
- **SSH, RDP and VNC sessions** in the app, through the server's Guacamole tunnel:
  - native CoreGraphics rendering (layers, PNG/JPEG/WebP images, remote cursor);
  - **trackpad** and **direct touch** modes, pinch to zoom, two-finger scroll wheel;
  - keyboard with Esc, Tab, Ctrl, Alt, arrows, F1–F12, and hardware keyboard support;
  - two-way clipboard, Ctrl+Alt+Delete, rotation with session resizing;
  - on-demand session recording.
- **Multiple sessions** open at once: minimize, session switcher, **workspaces** (groups of entries opened together).
- **Proxmox consoles** (VMs and containers) in the app, relayed by guacd on the server side.
- **Files** in sessions: SFTP over SSH, "Relay" drive over RDP (upload from Files, download, files pushed by Windows).
- **SSH command** across several servers: selection by folder, sudo, timeout, parallelism, library, live tracking, retry of failures, history.
- **Editing**: entries (create, edit, duplicate, delete), folders and their inherited credentials, vault, Markdown notes, attachments.
- **Administration**: Proxmox/NetBox sync sources, duplicates and merging, RDM import, encrypted backup (export and restore), availability retest.
- **Quick connection** over SSH/RDP without creating an entry.
- **Availability**: unreachable, reachable or unmeasured entries, retest on demand.
- **Vault**: username, password, SSH key and TOTP codes, behind Face ID; copy with expiration.
- **Recordings**: playback of recorded sessions (play, pause, ±10 s, speed ×0.5 to ×8, skipped pauses), deletion.
- Server **activity log**.

Web entries open in Safari.

## Server requirements

- A version of Relay that offers **Administration → Devices** (device tokens);
  for Proxmox consoles, `RELAY_BRIDGE_HOST` (the name by which guacd reaches Relay).
- If Relay sits behind SSO (oauth2-proxy…), the **`/api/`** path must be exempt from SSO:
  the app authenticates with `Authorization: Bearer`, without cookies. Relay checks every request itself.

## Security

- The token leaves the Keychain only for the `Authorization` header sent to the paired server.
- HTTPS required (HTTP only for `localhost`), redirects refused, ephemeral network session with no cookies or cache.
- Vault secrets are never stored: they are requested from the server at display time, after Face ID,
  and copied with a 90-second expiration, without iCloud sync.
- The app preview is hidden in the app switcher.
- Tests use only fictitious data (no real server).

## Architecture

| Folder | Role |
| --- | --- |
| `RelayKit/` | Swift package without UIKit: models, API client, pairing, Guacamole protocol. Tested on Linux and macOS. |
| `Relay/Core` | App session, Keychain, Face ID |
| `Relay/Features/Session` | Guacamole rendering (CoreGraphics), engine, remote screen, keyboard |
| `Relay/Features/*` | Pairing, connections, availability, vault, quick connection, log, settings |
| `RelayTests` / `RelayUITests` | Pixel-exact rendering tests and UI flows with screenshots |
| `scripts/` | Xcode project and icon generation, source check |

## Development

See [docs/INSTALLATION.md](docs/INSTALLATION.md): building with GitHub Actions and installing without a Mac.
Mapping to the server APIs: [docs/PARITE.md](docs/PARITE.md).

```bash
node scripts/generate-project.mjs     # after adding/removing a .swift file
node scripts/check-source.mjs         # before every commit
swift test --package-path RelayKit    # RelayKit tests (Linux or macOS)
```
