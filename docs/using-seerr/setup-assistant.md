---
title: Cross-platform setup assistant
description: Create a starter Docker Compose stack and connect its apps to SeerrNG.
sidebar_position: 5
---

# Cross-platform setup assistant

The setup assistant creates a Docker Compose stack and a per-stack guide for
the media services you choose. It runs on Windows, macOS, and Linux with Node.js
24.15.0 or newer in the 24.x line and Docker Compose v2. Docker Desktop is
suitable on Windows and macOS; Docker Engine with the Compose plugin is
suitable on Linux. The assistant does not install or configure Docker itself.

## Run the assistant

You can download the standalone assistant without cloning the full SeerrNG
repository. It uses Node.js built-ins and does not need an npm install.

On Windows, open PowerShell and run:

```powershell
Invoke-WebRequest https://raw.githubusercontent.com/snapetech/seerrng/main/scripts/setup-assistant.mjs -OutFile setup-assistant.mjs
node .\setup-assistant.mjs
```

On macOS or Linux, run:

```sh
curl --fail --location https://raw.githubusercontent.com/snapetech/seerrng/main/scripts/setup-assistant.mjs --output setup-assistant.mjs
node ./setup-assistant.mjs
```

You can also run the script directly from a SeerrNG source checkout:

```text
node scripts/setup-assistant.mjs
```

Choose a starter profile, confirm the output directory, and optionally let the
assistant start the stack after it writes the files. It writes files only; it
does not install apps, configure API keys, or start containers unless you
explicitly choose that option. To select a profile and output location
directly:

```text
node scripts/setup-assistant.mjs --profile movies-tv --output ./my-media-stack
```

For unattended file generation, add `--yes`; the assistant will write the
files but will not start containers. Add `--start` to explicitly start the
generated stack with Docker Compose.

If SeerrNG is already installed, generate only the companion apps on a shared
Docker network:

```text
node scripts/setup-assistant.mjs --profile movies-tv --apps -seerrng --network seerrng-shared --output ./media-apps
```

The generated guide includes the commands to create the shared network and
attach your existing SeerrNG container to it. The assistant does not scan
Docker networks or containers to guess which SeerrNG instance you mean. This
mode requires SeerrNG to run in a Docker container on the same Docker host;
native or remote SeerrNG installs need a separately reachable app address.

Use `--detect --output ./my-media-stack` to report which containers Docker
Compose currently sees in that generated stack. To discover supported apps
already attached to a particular Docker network, name that network explicitly:

```text
node scripts/setup-assistant.mjs --discover --network seerrng-shared
```

Add `--output ./connection-report` to write `CONNECTIONS.md` and
`seerrng-connections.json` with the detected app names, Compose hostnames,
container ports, and states. The assistant inspects only the named network and
refuses networks with more than 64 attached containers. It recognizes known
images and Compose service labels; custom images without a matching service
label may not be identified. The report contains no API keys or container
environment values. Enter each app's API key in SeerrNG, then use the existing
**Test** action to verify the connection and load its options before saving.

In SeerrNG, open **Settings → Services** and choose **Import detected
connections** to load `seerrng-connections.json`. Detected hostnames and ports
prefill matching add forms, including Prowlarr and software acquisition
settings. Saved service addresses are left unchanged. API keys remain blank;
enter each key in the corresponding app and use **Test** so SeerrNG validates
the API and loads profiles, root folders, or provider capabilities. Imported
suggestions stay in memory until you clear them or reload SeerrNG.

After choosing a profile, enter app IDs to add or prefix an ID with `-` to
remove it. For example, add `lidarr` to a Movies and TV profile or enter
`-qbittorrent` to omit its download client. Available app IDs are `seerrng`,
`radarr`, `sonarr`, `lidarr`, `bookshelf`,
`chaptarrng`, `prowlarr`, `qbittorrent`, `lazylibrarian`, `mylar3`, `kapowarr`,
`backissue`, `romarrng`, and `questarrng`.

Use `--list-profiles` to see the available profiles. Each generated folder
contains `compose.yaml` and `SETUP.md`. Review both before starting the stack.
The output files are never overwritten; choose another output directory when
you want to regenerate them.

## Starter profiles

| Profile | Included apps |
| --- | --- |
| `movies-tv` | SeerrNG, Radarr, Sonarr, Prowlarr, qBittorrent |
| `books` | SeerrNG, BookshelfNG, Prowlarr, qBittorrent |
| `books-chaptarrng` | SeerrNG, ChaptarrNG, Prowlarr, qBittorrent |
| `music` | SeerrNG, Lidarr, Prowlarr, qBittorrent |
| `comics` | SeerrNG, Mylar3 |
| `magazines` | SeerrNG, LazyLibrarian, Prowlarr, qBittorrent |
| `software` | SeerrNG, QuestarrNG, ROMarrNG, qBittorrent |
| `media-library` | SeerrNG, Radarr, Sonarr, Lidarr, BookshelfNG, Prowlarr, qBittorrent |
| `all-media` | SeerrNG, Radarr, Sonarr, Lidarr, BookshelfNG, Mylar3, LazyLibrarian, QuestarrNG, ROMarrNG, Prowlarr, qBittorrent |

These profiles are examples, not required SeerrNG dependencies. Choose a
smaller profile when you want fewer containers. BookshelfNG and ChaptarrNG are
alternatives for book automation; Mylar3, Kapowarr, and BackIssue are
alternatives for comics. The `all-media` profile chooses BookshelfNG and
Mylar3; add or remove providers with `--apps` for a different choice. For
example, replace BookshelfNG with ChaptarrNG using
`--profile books --apps -bookshelf,chaptarrng`, or add BackIssue as an
alternative with `--profile comics --apps -mylar3,backissue`. Connect only the
comic provider you want to use as the default destination. BackIssue currently
publishes a linux/amd64 image; on ARM64 hosts, choose Mylar3 or Kapowarr unless
you have enabled and verified x86 emulation.

The software profile includes both NG services because QuestarrNG handles PC
game requests and ROMarrNG handles emulation requests. Assign each supported
ROMarrNG system to Retro or Modern in SeerrNG's Software Acquisition settings.
If you need separate destinations, such as independent HD and 4K libraries,
add separately named services and give them distinct host ports and
configuration volumes in your Compose file.

## Link the apps

After starting a new stack, open SeerrNG at `http://localhost:5055` and
complete its first run. After discovering an existing network, open the URL for
your existing SeerrNG instance. In **Settings → Services**, add each supported media service and copy its
API key from that app. Configure Prowlarr under **Settings → Prowlarr** and
QuestarrNG or ROMarrNG under **Settings → Services → Software Acquisition**.
Select **Test** before saving; SeerrNG will validate the connection and load
the available profiles and root folders. Enter the Docker
Compose service name as the hostname (for example, `radarr`) when SeerrNG and
that app are in the generated stack. Use each app's listed container port, not
the host's published port. For BookshelfNG or ChaptarrNG, add two connections
to the same hostname and port: one with Book format and one with Audiobook
format. For emulation requests, assign supported ROMarrNG systems to **Retro**
or **Modern**.

Credentials are entered in SeerrNG's existing forms and are not generated,
copied between apps, or placed in the generated files. A successful connection
test does not save the service; review its options and save it explicitly.

## Storage and network behavior

The generated stack uses Docker named volumes for persistent app configuration,
media, and downloads when the selected apps need them. Media container paths
vary by app; the generated `SETUP.md` lists each mount so you can set matching
root folders. Apps in one generated stack share the `downloads` volume. Docker
manages where these volumes live on each operating system; back them up before
migrating the stack.

When qBittorrent is included, its first-start temporary `admin` password is
printed in its container log. Sign in at `http://localhost:8080` and set a
permanent password in the Web UI settings before connecting download clients.

Web ports bind to `127.0.0.1` so the interfaces are local to the Docker host by
default. When qBittorrent is included, it also publishes peer traffic on TCP
and UDP port `6881` on the Docker host for torrent transfers. To reach SeerrNG or an app's web
interface from another device, edit that service's host-side bind address after
reviewing your network and authentication setup. The app-to-app connections use
Compose DNS and remain inside the Compose network.

If one of the default host ports is already in use, set the corresponding
`*_HOST_PORT` variable in a `.env` file in the generated folder. The app-to-app
addresses and container ports in the guide do not change. For qBittorrent peer
traffic, set `QBITTORRENT_PEER_PORT` to change its host port.

The assistant does not configure indexers, download-client categories, media
root folders, quality profiles, TLS, or reverse proxies. Configure those in
the relevant app before relying on requests. ChaptarrNG is an alternative to
BookshelfNG; select one provider per format to avoid duplicate acquisitions.
After saving ROMarrNG, assign supported systems to **Retro** or **Modern** in
SeerrNG. Additional dashboards, monitoring, and update controllers remain
outside this acquisition-focused catalog.
