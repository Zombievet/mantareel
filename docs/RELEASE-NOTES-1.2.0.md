# Mantareel 1.2.0 — public beta

The first public release. Server 1.2.0 · Android app 2.2.0.

## Highlights

### Audio languages
Shows and films with several audio tracks — say Danish, English, Finnish,
Norwegian and Swedish — now **always start in English**, and every other
language is one tap away: the **Audio** menu in the browser, the **translate
button** on Samsung TVs and the **Audio language** button on Android. Tracks
are named properly ("English · 5.1", not "ENG"). If a file has no English
track, Mantareel starts on your device's language instead.

### Up next that waits for the end
The next episode is offered in the credits or the last 30 seconds, but never
starts until the episode you're watching has finished — then a 10-second
countdown. **Play now** skips it, **Stay here** stops at the end.

### Stay signed in
Devices you use stay signed in. Previously every phone, TV and browser had to
sign in again 30 days after it first did, however much it was used; now the
30 days count from the last time you used it. Signing a device out from
**Settings → Devices** still works instantly.

### Passwords
- **Forgot password?** on the sign-in page. Mantareel writes a one-time code to
  a file on your server — being able to open your own server's files is the
  proof it's you. Nothing is emailed or sent anywhere.
- **Change password** in Settings → Security.
- Admins can **set a new password** for anyone in Settings → Users.

## Fixes

- **Lip sync on Samsung TVs** — 25 fps shows (most European broadcasts) could
  drift out of sync when the server converted them. Video and audio are now
  cut on the same 4-second grid.
- **Paused streams** no longer vanish: the server used to end a stream after
  two idle minutes, so resuming a long pause froze for about a minute and a
  half. Streams now survive a pause, and a lost stream restarts instantly at
  the same moment.
- **The end of an episode on Samsung TVs** no longer shows "could not open
  the transcoded stream"; "Try again" at the end of an episode plays the next
  one rather than repeating the one you just watched.
- **Duplicate shows** — adding new episodes to a show and rescanning could
  file them under a second copy of the show. Fixed, and libraries already
  affected are repaired automatically when the server starts.
- **Android** — sign-in problems are explained in words ("Wrong username or
  password", "Can't reach the server") instead of "HTTP 401".
- Security: the web app's routing library is updated to clear two advisories.

## Downloads

| File | For |
|---|---|
| `ghcr.io/zombievet/mantareel:1.2.0` | Docker on Intel/AMD (amd64) and ARM (arm64) |
| `mantareel_1.2.0_amd64.deb` | Ubuntu 22.04+ / Debian 12+ on Intel/AMD |
| `mantareel_1.2.0_arm64.deb` | Raspberry Pi OS (64-bit), Ubuntu on ARM — *experimental* |
| `mantareel-setup-1.2.0.exe` | Windows 10/11 server |
| `mantareel-android-2.2.0.apk` | Android 7+ phones, tablets, NVIDIA Shield, Android TV |
| `mantareel-roku-1.6.zip` | Roku — *beta*, installed through the Roku's developer page |
| `mantareel-samsung-tv-3.7.4.zip` + `install-samsung-tv.ps1` | Samsung TVs 2021+ (Tizen 6.0+) — *beta*, signed for your TV with a free Samsung certificate |
| `SHA256SUMS.txt` | Checksums for every file |

**ARM is experimental:** it is built and tested in an emulator but not yet on
real Raspberry Pi hardware. Hardware transcoding on ARM boards isn't enabled
yet — those servers play files directly and convert in software.

## Upgrading

- **Docker:** change the tag to `1.2.0` and `docker compose up -d`. Your
  data folder is kept.
- **.deb:** `sudo apt install ./mantareel_1.2.0_amd64.deb` over the old one.
- **Windows:** run the new installer; it upgrades in place.
- **Android:** install the new APK over the old app; you stay signed in.
- **Roku / Samsung TV:** install the new version the same way as the first time.

Your library, users, watch history and settings carry over in every case.
Back up first if you like: Settings → Backups → **Run backup**.

## Known issues

- The Samsung TV and Roku apps are betas you install yourself; they are not in
  the Samsung or Roku stores yet. See the setup guide for both.
- On ARM the server converts video in software only (see above).
