# Mantareel setup guide

Everything from "I have some films in a folder" to watching on your TV and your
phone, at home and away. Take the steps in order the first time; afterwards,
jump to what you need.

**Contents**

1. [How it fits together](#1-how-it-fits-together)
2. [Install the server](#2-install-the-server)
   — [Docker](#2a-docker-recommended) ·
   [Synology](#synology-container-manager) ·
   [QNAP](#qnap-container-station) ·
   [Ubuntu / Debian](#2b-ubuntu--debian--raspberry-pi) ·
   [Windows](#2c-windows)
3. [First run](#3-first-run)
4. [Naming your files](#naming-your-files)
5. [Posters and summaries](#5-posters-and-summaries)
6. [People in your household](#6-people-in-your-household)
7. [Watching](#7-watching) — [browser](#in-a-browser) · [Android](#android) · [Roku](#roku-beta) · [Samsung TV](#samsung-tv-beta)
8. [Languages and subtitles](#8-languages-and-subtitles)
9. [Watching away from home](#9-watching-away-from-home)
10. [Hardware transcoding](#10-hardware-transcoding)
11. [Passwords](#11-passwords)
12. [Backups](#12-backups)
13. [Updating](#13-updating)
14. [Troubleshooting](#14-troubleshooting)

---

## 1. How it fits together

```
   your media folders ──► Mantareel server ──► browser / phone / TV
   (on a NAS or PC)        (runs at home)       (at home or away)
```

- **One server**, on a computer that stays on and can see your media: a NAS, a
  mini PC, a desktop, a Raspberry Pi 5.
- **Any number of apps** that connect to it: any web browser, the Android app,
  and the Roku and Samsung TV apps (beta).

Your media never leaves your server except to play on your own devices.
Mantareel has no cloud account and nothing to sign up for.

**Before you start, find your server's local IP address** — every app asks for
it. It looks like `192.168.1.20`. On Windows run `ipconfig`; on Linux
`hostname -I`; on a NAS it's in the network settings. This guide writes it as
`SERVER-IP`.

---

## 2. Install the server

Pick one.

### 2a. Docker (recommended)

Works on Linux, most NAS boxes, Unraid, TrueNAS, and Windows or Mac with
Docker Desktop. One image runs on Intel/AMD and ARM.

1. Make a folder for Mantareel, e.g. `~/mantareel`, and create
   `docker-compose.yml` in it:

   ```yaml
   services:
     mantareel:
       image: ghcr.io/zombievet/mantareel:1.2.0
       container_name: mantareel
       restart: unless-stopped
       ports:
         - "3099:3099"   # web app and API
       environment:
         TZ: "Europe/London"   # your time zone
       volumes:
         # Mantareel's own data: database, artwork, settings. Keep this.
         - ./mantareel-data:/var/lib/meridian
         # Your media, read-only. Add a line per folder.
         - /path/to/Movies:/media/movies:ro
         - /path/to/TV:/media/tv:ro
         - /path/to/Music:/media/music:ro
       devices:
         # Intel / AMD graphics for fast transcoding. Delete these two
         # lines if your machine has no /dev/dri (see section 10).
         - /dev/dri:/dev/dri
   ```

   Replace each `/path/to/...` with the real folder on your machine. The part
   after the colon (`/media/movies`) is what Mantareel sees — you'll type it
   in [first run](#3-first-run).

2. Start it:

   ```sh
   docker compose up -d
   ```

3. Open **`http://SERVER-IP:3099`** in a browser and go to
   [First run](#3-first-run).

**Check it's running:** `docker logs mantareel` should end with a line
containing `listening on http://0.0.0.0:3099`.

#### Synology (Container Manager)

1. Install **Container Manager** from Package Center.
2. In File Station, create `docker/mantareel/mantareel-data`.
3. Container Manager → **Project** → **Create**. Name it `mantareel`, path
   `docker/mantareel`, choose **Create docker-compose.yml**, and paste the file
   above. Use Synology paths for your media, e.g.
   `/volume1/video/Movies:/media/movies:ro`.
4. Keep the `devices` lines on models with Intel graphics (DS220+, DS224+,
   DS720+, DS920+). Remove them on models with no graphics chip — AMD Ryzen
   models such as the DS923+ and DS1522+, and ARM models; those convert video
   in software.
5. **Next → Done.** Open `http://SERVER-IP:3099`.

#### QNAP (Container Station)

1. Install **Container Station** from App Center.
2. In File Station, create `Container/mantareel`.
3. Container Station → **Applications** → **Create**, name it `mantareel`, and
   paste the file above with QNAP paths:
   - `./mantareel-data` → `/share/Container/mantareel`
   - your media, e.g. `/share/Multimedia/Movies:/media/movies:ro`
4. **Create.** Open `http://SERVER-IP:3099`.

Intel QNAP models (TS-x64, TS-x53D, TS-x73A …) transcode in hardware with the
`devices` lines kept.

### 2b. Ubuntu / Debian / Raspberry Pi

For Ubuntu 22.04 or newer, Debian 12 or newer, and Raspberry Pi OS (64-bit).

1. Download `mantareel_1.2.0_amd64.deb` (Intel/AMD) or
   `mantareel_1.2.0_arm64.deb` (Raspberry Pi 4/5 and other ARM boards —
   *experimental*) from the [latest release](../../../releases/latest).
2. Install it:

   ```sh
   sudo apt install ./mantareel_1.2.0_amd64.deb
   ```

   This creates a `meridian` service that starts at boot, keeps its data in
   `/var/lib/meridian`, and bundles everything it needs (ffmpeg included).
3. Let Mantareel read your media. It runs as the `meridian` user, which needs
   read access to your folders:

   ```sh
   sudo setfacl -R -m u:meridian:rX /path/to/Movies /path/to/TV
   ```

   (or add `meridian` to a group that can read them).
4. Open **`http://SERVER-IP:3099`** and go to [First run](#3-first-run).

**Useful commands**

```sh
sudo systemctl status meridian      # is it running?
sudo systemctl restart meridian     # restart
sudo journalctl -u meridian -f      # live log
```

Settings that aren't in the web app live in `/etc/meridian/meridian.env`;
restart the service after changing it.

### 2c. Windows

For Windows 10 or 11, on the PC that holds your media.

1. Download `mantareel-setup-1.2.0.exe` from the
   [latest release](../../../releases/latest).
2. Right-click it → **Run as administrator**, and follow the steps. It installs
   Mantareel as a Windows service that starts with the PC, opens port 3099 on
   your home network (never on public networks), and bundles ffmpeg.
3. When it finishes it opens **`http://localhost:3099`**. Go to
   [First run](#3-first-run). From other devices, use `http://SERVER-IP:3099`.

Your data lives in `C:\ProgramData\Meridian`. To stop or start the server:
Start menu → **Mantareel** → **Stop Mantareel service** / **Start Mantareel
service**.

> **Windows SmartScreen** may say it "protected your PC" because the installer
> is new. Click **More info → Run anyway**. Compare the file's checksum with
> `SHA256SUMS.txt` in the release if you want to be sure it's genuine:
> `Get-FileHash .\mantareel-setup-1.2.0.exe` in PowerShell.

---

## 3. First run

1. Open **`http://SERVER-IP:3099`**. You'll see **First-run setup**.
2. Choose a **username** and **password** for the administrator — you. Email is
   optional and never leaves your server. Write the password down; if you lose
   it, see [Passwords](#11-passwords).
3. You're in. Open **Settings → Libraries → Add library** and add one library
   per kind of media:

   | Field | What to enter |
   |---|---|
   | Name | Anything — "Movies", "Kids' films", "Concerts" |
   | Type | **Movies**, **TV Shows**, **Music** or **Photos** |
   | Folder | The path **as Mantareel sees it** |

   **Docker:** the right-hand side of your volume line — `/media/movies`,
   `/media/tv`.
   **Linux .deb:** the real path, e.g. `/srv/media/Movies`.
   **Windows:** the real path, e.g. `D:\Media\Movies`, or a network share like
   `\\NAS\Media\Movies`.

4. Mantareel scans the folder straight away. A few hundred films take a minute
   or two; artwork keeps arriving for a while after that. New files are picked
   up automatically.

---

## Naming your files

Mantareel recognises the common layouts used by other media servers. If yours
already works with Plex or Jellyfin, it will work here.

**Films** — one folder per film is best:

```
Movies/
  Heat (1995)/
    Heat (1995).mkv
  The Iron Giant (1999)/
    The Iron Giant (1999).mp4
```

**TV shows** — show folder, then season folders, with `S01E01` in each name:

```
TV/
  Be Cool, Scooby-Doo! (2015)/
    Season 01/
      Be Cool, Scooby-Doo! (2015) - S01E01 - Mystery 101.mkv
      Be Cool, Scooby-Doo! (2015) - S01E02 - Party Like It's 1899.mkv
```

**Music** — `Artist/Album/01 - Track.flac` (tags are read too).

**Subtitles** — next to the video, with the language before the extension:
`Heat (1995).en.srt`, `Heat (1995).da.srt`, `Heat (1995).en.forced.srt`.

Quality tags in names (`1080p`, `BluRay`, `x265`) are ignored, so you don't
need to rename downloads that already include them.

---

## 5. Posters and summaries

Mantareel fetches posters, summaries, cast and ratings from
[The Movie Database (TMDB)](https://www.themoviedb.org). It needs a free API
key of your own:

1. Create a free account at themoviedb.org.
2. Go to **Settings → API** on their site and request a key (choose
   *Developer*, personal use).
3. In Mantareel, open **Settings → Metadata**, paste the key into
   **TMDB API key**, and save.
4. **Settings → Libraries → Refresh metadata for all libraries.**

Without a key, Mantareel still plays everything and uses any `.nfo` files and
artwork already in your folders.

**Wrong match?** Open the title, choose **Fix match**, and pick the right one.
**Settings → Metadata** lists any films it couldn't identify.

---

## 6. People in your household

**Settings → Users.**

- **Add user** — fill in their name and what they may see, then share the
  **activation link**. They open it, choose their own password, and they're in.
  The link works once and expires.
- **Add kid user** — a profile with an age limit; only films and shows rated
  for that age appear.
- **Library access** — limit any user to certain libraries.
- **Profile PIN** — a 4-digit PIN asked for on *Who's watching* before anyone
  can switch into that profile.

Each person gets their own continue-watching, watchlist, recommendations and
watched marks.

**Devices.** **Settings → Devices** lists every browser, phone and TV signed
in to your account. **Revoke** signs one out immediately — use it for a lost
phone.

Devices stay signed in while they're used; a device left unused for 30 days
has to sign in again.

---

## 7. Watching

### In a browser

Open `http://SERVER-IP:3099` in Chrome, Edge, Firefox or Safari on any
computer, tablet or phone. Nothing to install.

Mantareel plays the original file when the browser can, and converts on the
fly when it can't (for example 4K HDR on a laptop, or DTS audio).

**Keyboard:** Space play/pause · ← → 10 seconds · F full screen · M mute.

### Android

For phones, tablets, NVIDIA Shield and Android TV (Android 7 or newer).

1. On the device, download `mantareel-android-2.2.0.apk` from the
   [latest release](../../../releases/latest).
2. Open it. Android asks to allow installs from your browser or file app —
   allow it, then **Install**.
3. Open Mantareel, enter `http://SERVER-IP:3099`, and sign in.

On Android TV / Shield without a browser: install **Downloader** from the Play
Store and enter the APK's download link.

The app can **download** films and episodes for offline viewing
(title → **Download**), and has its own Audio language, subtitle, dialogue
boost and night-mode controls.

### TV apps

Both TV apps are **beta** downloads you install yourself, until they reach the
Samsung and Roku stores. Both sign in **with your phone**: the TV shows a QR
code; scan it with a phone that's signed in to Mantareel and the TV is signed
in. No typing with a remote.

#### Roku (beta)

About ten minutes, no extra software.

1. **Turn on Developer Mode.** With the Roku remote press
   **Home ×3, Up ×2, Right, Left, Right, Left, Right**. A *Developer Settings*
   screen appears: note the address it shows (e.g. `http://192.168.1.60`),
   choose **Enable installer and restart**, accept the agreement, and set a
   password. The Roku restarts.
2. On a computer or phone on the same network, open that address in a
   browser. Sign in as `rokudev` with the password you just set.
3. **Upload** `mantareel-roku-1.6.zip` from the
   [latest release](../../../releases/latest) and press **Install**.
4. Mantareel opens on the Roku. Enter your server's address
   (`http://SERVER-IP:3099`) and sign in with your phone.

A Roku holds one self-installed app at a time; Mantareel stays until you
install a different one or turn Developer Mode off. To update, upload the new
zip the same way.

#### Samsung TV (beta)

Samsung only lets a TV run self-installed apps that are signed for that exact
TV, so the first install has some one-time setup on a Windows, Mac or Linux
computer (about 30 minutes). Updates afterwards take a minute. It needs Tizen 6.0 or
newer: most Samsung TVs from 2021 onward.

**One-time setup**

1. **Find the TV's IP address:** Settings → General → Network → Network
   Status → IP Settings.
2. **Developer Mode on the TV:** open **Apps**, then type **1 2 3 4 5** on the
   remote (or the on-screen number pad). Switch **Developer mode** on, enter
   **your computer's** IP address, then turn the TV off and on again by
   holding the power button.
3. **Install Tizen Studio** (free, from Samsung:
   developer.samsung.com/smarttv → Tools → Tizen Studio). When it opens the
   **Package Manager**, install **Tizen SDK tools**, and under
   **Extension SDK** install **TV Extensions** and
   **Samsung Certificate Extension**.
4. **Connect the TV in Tizen Studio:** Tools → **Device Manager** → *Remote
   Device Manager* → **+**, enter the TV's IP, and switch it on. The TV appears
   in the list.
5. **Create your certificate:** Tools → **Certificate Manager** → **+** →
   **Samsung** → **TV**. Give it a name (e.g. `MyTV`), create an *author
   certificate* (sign in with a free Samsung account when asked), then a
   *distributor certificate* — keep **Public** privilege, and the connected
   TV's ID (DUID) fills in automatically. Add every Samsung TV you want to use.

**Install**

Download `mantareel-samsung-tv-3.7.4.zip` and `install-samsung-tv.ps1` from
the [latest release](../../../releases/latest) into the same folder.

*Windows* — open PowerShell in that folder and run (with your TV's IP):

```powershell
powershell -ExecutionPolicy Bypass -File .\install-samsung-tv.ps1 -TvIp 192.168.1.50
```

It signs the app with your certificate, installs it and opens it on the TV.

*Mac or Linux* — in a terminal, using the `tizen` and `sdb` tools from Tizen
Studio (`~/tizen-studio/tools/ide/bin` and `~/tizen-studio/tools`):

```sh
mkdir mantareel-tv && unzip mantareel-samsung-tv-3.7.4.zip -d mantareel-tv
tizen package -t wgt -s MyTV -- mantareel-tv
sdb connect 192.168.1.50:26101
tizen install -n Mantareel.wgt -s 192.168.1.50:26101 -- mantareel-tv
tizen run -p MeridianTV.Meridian -s 192.168.1.50:26101
```

On the TV, enter your server's address (`http://SERVER-IP:3099`) and sign in
with your phone.

**Good to know**

- If the install is refused with a certificate error, the certificate doesn't
  include that TV: add its DUID in Certificate Manager and run the install
  again.
- Self-installed apps can stop opening after some weeks in Developer Mode.
  Run the install again to refresh it; your sign-in and settings are kept.
- Leave the TV on its Home screen while installing.

---

## 8. Languages and subtitles

**Audio.** When a film or episode has several audio languages, Mantareel
**starts in English** and every other language is in the player's **Audio**
menu (browser), **translate** button (TV) or **Audio language** button
(Android). If there's no English track it starts in your device's language.

**Subtitles.** The **CC** button lists every subtitle in the file and next to
it. Going back 10 seconds with subtitles off turns them on for the bit you
missed, then off again.

**Missing subtitles?** **Settings → Subtitles** can fetch them from
OpenSubtitles (free account), and an admin can switch on subtitle generation,
which listens to the audio and writes subtitles on your own server.

---

## 9. Watching away from home

Mantareel is **private by default**: nothing outside your home can reach it
until you choose how. **Settings → Remote Access** walks through each option on
your own server; in short:

| Option | Best for | What you need |
|---|---|---|
| **WireGuard** (direct) | Most private; your devices join your home network | A router that supports port forwarding, or a WireGuard-capable router |
| **Tailscale** (private) | Easiest private option; works behind any router | A free Tailscale account; the Tailscale app on each device |
| **Public access** (Tailscale Funnel or Cloudflare tunnel) | Sharing with people who won't install anything | Understand that anyone with the address reaches your sign-in page |
| **Your own reverse proxy** (Caddy, nginx, Traefik) | People who already run one | A domain and HTTPS certificate |

Remote viewers automatically get a stream sized to their connection, with a
deep buffer that rides out mobile dropouts.

---

## 10. Hardware transcoding

When a device can't play a file as it is, Mantareel converts it while you
watch. With hardware help, one server converts several 4K streams at once;
without, it converts in software, which is fine for 1080p but slow for 4K.

**Check what you have:** the admin **Dashboard**
(`http://SERVER-IP:3099/admin`) shows the encoder in use (for example `h264_qsv` = Intel Quick Sync, `h264_vaapi` = Intel/AMD,
`h264_nvenc` = NVIDIA, `libx264` = software).

| Hardware | Docker | Linux .deb | Windows |
|---|---|---|---|
| **Intel** (Quick Sync, 7th gen or newer) | keep `devices: /dev/dri` | automatic | automatic |
| **AMD** graphics / APU | keep `devices: /dev/dri` | automatic | automatic |
| **NVIDIA** | install the NVIDIA Container Toolkit, then add `gpus: all` under the service | install the NVIDIA driver | automatic with a current driver |
| **ARM** (Raspberry Pi …) | software only in 1.2.0 | software only | — |

If Docker can't open `/dev/dri`, the server falls back to software and says so
in its log (`docker logs mantareel`).

---

## 11. Passwords

**Change your own password:** **Settings → Security → Password.** Your other
devices are signed out; the one you're using stays signed in.

**Someone else forgot theirs:** an admin opens **Settings → Users**, finds the
person, and uses **Set a new password**.

**You forgot yours (even the admin):**

1. On the sign-in page, choose **Forgot password?**, type your username, and
   press **Write a reset code**.
2. Mantareel writes a one-time code to a file called `password-reset.txt` in
   its data folder. Open that file on your server:

   | Install | File |
   |---|---|
   | Docker | `mantareel-data/password-reset.txt` (next to your `docker-compose.yml`) |
   | Synology / QNAP | the data folder you created, via File Station |
   | Linux .deb | `sudo cat /var/lib/meridian/password-reset.txt` |
   | Windows | `C:\ProgramData\Meridian\password-reset.txt` |

3. Type the code and your new password. The code works once, for 15 minutes,
   and you're signed out everywhere else.

Being able to open your server's files is the proof it's you — nothing is
emailed and nothing goes over the internet.

On Docker you can also print a code from a terminal:
`docker exec mantareel meridian reset-code YOUR-USERNAME`.

---

## 12. Backups

**Settings → Backups.**

- **Run backup** saves the database, artwork and settings to one `.tar.gz`
  file. Your media isn't included — it's already on your disks.
- **Schedule** a daily or weekly backup to a folder, and keep the last few.
- **Restore from archive** brings a backup back, for example on a new server.

Keep a copy of the backup somewhere other than the server's own disk.

---

## 13. Updating

Updates keep your library, users, history and settings.

- **Docker:** change the image tag (e.g. `:1.2.0` → `:1.3.0`), then
  `docker compose pull && docker compose up -d`.
- **Linux .deb:** `sudo apt install ./mantareel_<version>_amd64.deb`.
- **Windows:** run the new installer.
- **Android:** install the new APK over the app.

**Settings → About** shows your version. New releases appear on the
[releases page](../../../releases).

---

## 14. Troubleshooting

**The page won't open at `http://SERVER-IP:3099`.**
- Is the server running? (`docker ps`, `systemctl status meridian`, or the
  Windows service *MeridianMediaServer*.)
- Is it the right IP, and is your device on the same network?
- A firewall on the server may block port 3099. On Linux with ufw:
  `sudo ufw allow 3099/tcp`.

**A library is empty after adding it.**
- Docker: did you type the *container* path (`/media/movies`), not the
  host path? Is the folder in your `volumes:` list?
- Linux: can the `meridian` user read the folder? (see
  [2b](#2b-ubuntu--debian--raspberry-pi), step 3)
- Windows network shares: the service runs as the local system account, which
  may not have the share's password. Map the share for that account or copy
  the media locally.

**No posters.** Add a TMDB key ([section 5](#5-posters-and-summaries)) and
refresh metadata. Check file names ([Naming your files](#naming-your-files)).

**Playback stutters or keeps buffering.**
- Away from home: the connection may be too slow for the original file; the
  player steps down quality automatically after a few stalls. Pick a lower
  quality in the player's **Quality** menu.
- At home: check the **Dashboard** (`http://SERVER-IP:3099/admin`). If
  *Server too slow* is
  rising, the server can't convert fast enough — set up
  [hardware transcoding](#10-hardware-transcoding).

**Wrong audio language or no sound.** Pick the language in the player's
Audio / translate menu ([section 8](#8-languages-and-subtitles)). On a TV
connected to a receiver, try the other audio track if one is DTS.

**Signed out on a device.** Devices stay signed in while used; after 30 days
unused, sign in again. If a device was revoked in Settings → Devices it must
sign in again.

**Something else.** **Settings → Report a problem**, or
[open an issue](../../../issues/new) with your version (Settings → About),
what you were doing, and what you saw.
