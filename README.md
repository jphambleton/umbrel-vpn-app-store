# VPN Apps — Umbrel Community App Store

Apps that route their traffic through a VPN tunnel using [Gluetun](https://github.com/qdm12/gluetun).

## Apps

- **vpn-transmission** — Transmission BitTorrent client, all traffic forced through AirVPN (WireGuard) with a kill switch.

## Install the store

umbrelOS → App Store → `⋯` (top right) → Community App Stores → paste this repo's GitHub URL → Add.

## Configure vpn-transmission (do this BEFORE clicking Install, or install then restart)

The VPN keys are **not** stored in this repo. They live in files inside the app's data directory.

1. AirVPN Client Area → **Config Generator** → OS: *Linux* → Protocol: *WireGuard* → pick a server/country → Generate. Open the `.conf`.
2. Reserve a forwarded port at https://airvpn.org/ports/ and write it into `vpn-transmission/docker-compose.yml`
   in both places marked `# <-- AirVPN forwarded port` (seeding works badly without it).
3. Install the app from the store. It will fail to connect (no keys yet) — that's expected.
4. umbrelOS → Files → Apps → VPN Transmission → `secrets/` (already exists, has a README) and create three plain-text files (no trailing newline needed, Gluetun trims):
   - `wireguard_private_key`  — value of `PrivateKey` from the .conf
   - `wireguard_preshared_key` — value of `PresharedKey` from the .conf
   - `wireguard_addresses`     — the **IPv4 only** part of `Address`, e.g. `10.170.23.45/32` (drop the IPv6 one; Umbrel Docker has no IPv6)
5. Restart the app (right-click tile → Restart).
6. Check Gluetun's logs: they should end with `Public IP address is <AirVPN IP>` and `healthy!`.

## Point Radarr / Sonarr / etc. at it

Download client host: `vpn-transmission_gluetun_1`, port `9091`. Paths stay `/downloads` — same host folder as the official Transmission app.

## Change country

Edit `SERVER_COUNTRIES` in `docker-compose.yml`, commit, then update the app in umbrelOS (or restart the app if the store refreshed).

## Known caveats

- If the Gluetun container restarts on its own, Transmission loses its network namespace and must be restarted too. Restarting the whole app from the umbrelOS tile fixes it.
- Every commit that changes `docker-compose.yml` should bump `version` in `umbrel-app.yml` so umbrelOS offers the update.
