# Self-Hosted Services — Complete Stack Reference

## The SaaS replacement map

| You pay for | You replace it with | Self-hosted app | Difficulty |
|---|---|---|---|
| Spotify | Your music library | Navidrome | Easy |
| Google Drive | Your files | NextCloud | Medium |
| Google Docs | Collaborative editing | NextCloud + Collabora | Medium |
| Google Calendar | Calendar sync | NextCloud Calendar | Easy |
| Google Contacts | Contact sync | NextCloud Contacts | Easy |
| Google Photos | Photo backup | NextCloud Photos or Immich | Medium |
| Discord | Group chat | Matrix + Element | Medium |
| WhatsApp | Messaging | Matrix (with bridges) | Medium |
| Dropbox | File sync | NextCloud | Medium |
| GitHub (private) | Code hosting | Gitea | Medium |
| Notion | Notes + docs | Outline or Joplin Server | Medium |
| Pocket/Instapaper | Read-later | Wallabag | Easy |
| Google Search | Web search | SearXNG | Easy |
| Cloudflare 1.1.1.1 DNS | DNS resolution | Pi-hole + Unbound | Medium |
| Bitwarden (paid) | Password manager | Vaultwarden | Easy |

---

## CasaOS — your app control plane

CasaOS runs on any machine with Docker. Install it once and manage all services from
a browser UI. Every app below can be installed via CasaOS's app store in 2 clicks.

```bash
curl -fsSL https://get.casaos.io | sudo bash
```

Access CasaOS at `http://<your-node-tailscale-ip>:80`

CasaOS handles: Docker Compose, persistent storage, automatic restarts, update
notifications, and resource monitoring. You rarely need to touch YAML files.

---

## NextCloud — files, calendar, contacts, photos

### Installation (NextCloud AIO recommended)

```bash
# Via CasaOS: Apps → Community Apps → NextCloud AIO
# Or manually:
docker run -d \
  --name nextcloud-aio-mastercontainer \
  --restart always \
  -p 8080:8080 \
  -e APACHE_PORT=11000 \
  -v nextcloud_aio_mastercontainer:/mnt/docker-aio-config \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  nextcloud/all-in-one:latest
```

Access setup at `https://<tailscale-ip>:8080`

NextCloud AIO includes: NextCloud core, Collabora Online (document editing),
Nextcloud Talk (video calls), ClamAV (virus scanning), Imaginary (image processing),
Fulltextsearch (file search). Everything in one Docker stack.

### Client apps per person

| Platform | App | Notes |
|---|---|---|
| Windows / macOS | NextCloud Desktop | Official, works well |
| Android | NextCloud app | File sync + auto photo upload |
| iOS | NextCloud app | Same |
| Calendar (any) | Any CalDAV client | Built into iOS/macOS/Thunderbird |
| Contacts (any) | Any CardDAV client | Built into iOS/macOS/Thunderbird |

### Photo backup setup

Configure each person's phone to auto-upload photos to NextCloud. Photos stay on
your hardware, not Google Photos or iCloud. The NextCloud iOS/Android apps have
auto-upload built in.

```
NextCloud Settings → Mobile devices → Auto-upload
Set upload folder: /Photos/<your-name>/
Enable: Upload on WiFi only (saves data)
```

---

## Navidrome — music streaming (Spotify replacement)

### What it is

Navidrome indexes your music library and serves it via the Subsonic API. Any Subsonic-
compatible app becomes your Spotify alternative. The group's collective library is
available to everyone.

### Installation

```yaml
# docker-compose.yml
services:
  navidrome:
    image: deluan/navidrome:latest
    restart: unless-stopped
    ports:
      - "4533:4533"
    volumes:
      - /your/music/library:/music:ro
      - navidrome_data:/data
    environment:
      ND_SCANSCHEDULE: "1h"
      ND_LOGLEVEL: "info"
      ND_SESSIONTIMEOUT: "24h"
      ND_BASEURL: ""
```

### Music library organization

Everyone contributes to the shared music folder. A simple convention:

```
/music/
├── library/
│   ├── Artist Name/
│   │   ├── Album Name (Year)/
│   │   │   ├── 01 - Track Name.flac
│   │   │   └── cover.jpg
```

Use beets (https://beets.io) to auto-organize and tag your existing library before
importing. Running `beet import /your/music` will sort everything into the right
structure automatically.

### Client apps

| Platform | App | Free? |
|---|---|---|
| Android | Symfonium | Paid (~$5, worth it) |
| Android | Subsonic (DSub) | Free |
| iOS | Amperfy | Free |
| iOS | play:Sub | Paid |
| Desktop | Feishin | Free, open source |
| Web | Navidrome web UI | Built-in |

### Playlist sharing

Navidrome has a built-in playlist system. Create a shared playlist, add it to a
shared account, and everyone in the group can follow it. Works like Spotify's
collaborative playlists.

---

## Matrix / Element — group communication

### Why Matrix over Signal or Telegram

Signal: excellent encryption, but centralized. Signal's servers or phone number
requirement can be a single point of failure.

Telegram: not end-to-end encrypted by default, centralized Russian company.

Matrix: federated protocol. Your homeserver talks to any other Matrix server.
Even if your server goes down, everyone's messages are preserved on their own
clients. Bridges exist for Signal, Telegram, WhatsApp, Discord.

### Synapse homeserver installation

```bash
# Generate config
docker run -it --rm \
  -v ~/synapse:/data \
  -e SYNAPSE_SERVER_NAME=matrix.your-tailscale-hostname \
  -e SYNAPSE_REPORT_STATS=no \
  matrixdotorg/synapse:latest generate

# Run
docker run -d \
  --name synapse \
  --restart unless-stopped \
  -p 8448:8448 \
  -v ~/synapse:/data \
  matrixdotorg/synapse:latest
```

### Client apps

| Platform | App | Notes |
|---|---|---|
| All platforms | Element | Official, feature-complete |
| Android | Schildichat | Element fork, better UX |
| iOS | Element X | New faster client |
| Desktop | Cinny | Clean web-based client |
| Desktop | Beeper | Aggregates multiple platforms |

### Bridges for people who won't switch

Matrix bridges let your homeserver relay messages to/from other platforms.
The person on Matrix talks to the person on WhatsApp without either switching apps.

Popular bridges:
- mautrix-whatsapp: WhatsApp bridge
- mautrix-telegram: Telegram bridge
- mautrix-signal: Signal bridge  
- matrix-appservice-discord: Discord bridge

---

## Immich — Google Photos replacement

Navidrome is for music. Immich is for photos. It has face recognition, location
mapping, album sharing, and mobile apps that auto-backup your camera roll.

```yaml
services:
  immich-server:
    image: ghcr.io/immich-app/immich-server:release
    restart: always
    ports:
      - "2283:2283"
    volumes:
      - /your/photos:/usr/src/app/upload
      - /etc/localtime:/etc/localtime:ro
```

---

## Vaultwarden — password manager (Bitwarden compatible)

Self-hosted Bitwarden server. Use any Bitwarden client app. Your group's passwords
stay on your hardware.

```bash
docker run -d \
  --name vaultwarden \
  -p 8082:80 \
  -v vaultwarden_data:/data \
  vaultwarden/server:latest
```

Point the Bitwarden app to your Tailscale IP instead of bitwarden.com.

---

## Gitea — GitHub replacement

```bash
docker run -d \
  --name gitea \
  -p 3000:3000 \
  -p 222:22 \
  -v gitea_data:/data \
  gitea/gitea:latest
```

Gitea UI is almost identical to GitHub. Includes pull requests, issues, wikis,
project boards, and CI/CD via Woodpecker CI (a separate service that integrates
natively with Gitea).

---

## Pi-hole + Unbound — DNS independence

Pi-hole blocks ads and trackers at the network level. Unbound resolves DNS directly
from root servers without sending queries to Google or Cloudflare.

```bash
# Pi-hole
curl -sSL https://install.pi-hole.net | bash

# Unbound
sudo apt install unbound

# Configure Pi-hole to use Unbound as upstream (127.0.0.1#5335)
# See: https://docs.pi-hole.net/guides/dns/unbound/
```

Set Pi-hole as the DNS server in your router (or on each device) and every device
on your Tailscale network gets ad blocking and private DNS.
