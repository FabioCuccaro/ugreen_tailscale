# Tailscale on a UGREEN NAS (UGOS, Docker)

Reach the services on your UGREEN NAS (UGOS web UI, shared folders, Plex, your own containers) from anywhere, **without exposing anything to the internet**: no ports opened on the router, no UGOS *Quick access* (`…ugdocker.link`) and no UGREENlink.

The NAS joins your [Tailscale](https://tailscale.com) network (tailnet) through a single Docker container started from the UGOS **Docker** app. Your phone and computers run the Tailscale app with the same account, and the NAS shows up as `nas`.

🇮🇹 **Italiano:** [guida passo passo](docs/it/guida.md) · [checklist da telefono](docs/it/checklist.md) · [notifiche con ntfy](docs/it/ntfy.md)

## Who this is for

- You have a UGREEN NAS running **UGOS** (Pro) and you manage containers from the web UI, with **Docker → Project**. No SSH needed.
- You want to open your NAS services from your phone or laptop when you are away from home.
- You don't want to rely on UGREEN's relay services or open ports on your router.

## How it works

```
 phone / laptop ──(Tailscale, WireGuard, end-to-end encrypted)──► NAS (container tailscale, host network)
   http://nas:<port>                                                   └─ every service on the NAS
```

- One `tailscale/tailscale` container with `network_mode: host`: the NAS itself gets a `100.x.y.z` address in your tailnet, so **every port of the NAS** is reachable through it, not just one container.
- Traffic is encrypted end to end (WireGuard). Tailscale's coordination servers know which devices connect, but cannot see your data.
- Nothing listens on the internet. If your phone is not on your tailnet, the NAS is unreachable from outside.

Two compose files are provided:

| File | When | Status |
|---|---|---|
| [`docker-compose.yml`](docker-compose.yml) | **Plan A**, kernel networking (`/dev/net/tun`, `cap_add`). Use this first | ✅ Works on a DXP2800 with UGOS (October 2026) |
| [`docker-compose.userspace.yml`](docker-compose.userspace.yml) | **Plan B**, userspace networking. Only if UGOS refuses `devices` / `cap_add` or `/dev/net/tun` is missing | ⚠️ Not yet tested on a real NAS |

## Setup

### 1. Turn off UGREENlink (recommended)

The UGOS desktop address `ug.link/<ID>` and the container *Quick access* addresses (`…ugdocker.link`) both go through **UGREENlink**, UGREEN's relay. On a DXP2800 (October 2026) the Docker app has **no per-container switch**: the switch is for the whole NAS.

1. **Control Panel → Remote access → Method 1** → untick **UGREENlink remote access** and save. If a *Method 2* (port forwarding, DDNS) is enabled, turn it off too.
2. From your phone, **with Wi‑Fi and Tailscale off**: `ug.link/<ID>` and any old `…ugdocker.link` address must no longer open.
3. If a service was exposed that way, change its password.

From then on you reach UGOS and the UGREEN app through Tailscale (see [Reaching your services](#reaching-your-services)). If you are away from home and rely on UGREENlink right now, do this step once Tailscale works (step 4).

### 2. Tailscale account and auth key

1. Sign in at <https://login.tailscale.com> with the account you **already** use on your phone and computers. A new account would put the NAS in a different tailnet, invisible to your other devices.
2. Optional: *Settings → Keys → Generate auth key*, **not** reusable, **not** ephemeral. Copy it somewhere safe: it is shown only once. You can also skip the key and log in with a link (step 3).

### 3. Create the `tailscale` Project in UGOS

1. **Docker → Project → Create**. Name: `tailscale`. Folder: create `tailscale` inside your `docker` shared folder.
2. Paste [`docker-compose.yml`](docker-compose.yml) and replace `<tailscale auth key>` with your key.
   *No key?* Delete the `TS_AUTHKEY` line, start the Project, open the container **Logs** and open the `https://login.tailscale.com/a/…` link printed there.
3. Start it. The Logs must not show errors about `/dev/net/tun` or permissions.
4. On <https://login.tailscale.com/admin/machines> the machine `nas` appears with a `100.x.y.z` address.
5. On the `nas` row: `…` menu → **Disable key expiry**. Otherwise the NAS drops out of your tailnet after a few months.
6. If you used a key, delete the `TS_AUTHKEY` line and **Redeploy**: the node stays logged in (its state lives in `docker/tailscale/state`).

**If UGOS refuses the compose** or the Logs mention `/dev/net/tun`, `devices` or `cap_add`: replace the whole text with [`docker-compose.userspace.yml`](docker-compose.userspace.yml) (plan B), generate a new key if the first one was used, and start again from point 4. Please open an issue with the exact error and your NAS model.

### 4. Test from your phone

1. Install the Tailscale app (Android, iOS) and sign in with the same account.
2. **Turn Wi‑Fi off** and open `http://nas:<port>` for one of your services (for example the UGOS web UI at `https://nas:9443`, or a container's port).
3. Turn Tailscale off in the app: the same address must stop answering.

On Android you can make Tailscale an *Always-on VPN* (Settings → Network & internet → VPN → gear icon next to Tailscale) so it never turns itself off.

### 5. Other devices

Install Tailscale on Windows, macOS or Linux from <https://tailscale.com/download> with the same account. On Linux you can check the link with `tailscale ping nas`.

## Reaching your services

With the Tailscale app on:

| Service | Address |
|---|---|
| Any container with a published port | `http://nas:<port>` (or `http://100.x.y.z:<port>`) |
| UGOS web UI (replaces `ug.link`) | `https://nas:9443` or `http://nas:9999` (UGOS default ports; if you changed them, use yours). With `https` the browser may warn about the certificate: traffic is encrypted by Tailscale anyway |
| UGREEN app on the phone | Add the NAS by address, `https://nas:9443`, with Tailscale on (✅ Android) |
| Shared folders (SMB) | `\\nas\<share>` on Windows, `smb://nas/<share>` on macOS / Linux / phone apps |

If the name `nas` doesn't resolve, check that *MagicDNS* is enabled in the Tailscale admin console (*DNS* page), or use the `100.x.y.z` address.

## Optional: use the NAS's home address from outside

Some apps store the NAS's LAN address (for example `\\192.168.1.10\share` in a Power BI report or a mapped drive). To make that **same address** work away from home, announce a route for **the NAS only**:

1. Reserve the NAS's IP in your router's DHCP settings, so it never changes.
2. In the compose, uncomment `TS_ROUTES: "<NAS LAN IP>/32"`, put the NAS IP, and **Redeploy**.
3. Admin console → *Machines* → `nas` → *Subnets* → **Edit** → tick the route → **Save**.
4. Windows, iOS and Android accept routes automatically; on Linux run `sudo tailscale set --accept-routes`.

Announce only the `/32` of the NAS, never your whole home network (`192.168.1.0/24`). IP forwarding on the NAS is not needed for a `/32` that is the NAS's own address: Tailscale skips that check (source, `net/netutil/ip_forward.go`). ⚠️ Not yet tested end to end.

## Optional: push notifications with ntfy

[`docker/ntfy/docker-compose.yml`](docker/ntfy/docker-compose.yml) runs a private [ntfy](https://ntfy.sh) server on the NAS, reachable only through Tailscale (`http://nas:8090`), with login required and no Firebase. Scripts and containers on the NAS can then send notifications to your phone. Guide: [`docs/ntfy.md`](docs/ntfy.md) (Italian: [`docs/it/ntfy.md`](docs/it/ntfy.md)). ✅ Tested on an Android phone over mobile data.

The Project [`docker/watchdog`](docker/watchdog/docker-compose.yml) uses it to notify you when a container crashes, stays stopped or turns unhealthy ([guide](docs/ntfy.md#container-down-alerts)).

## Security notes

- **Never commit secrets**: the filled-in compose (auth key), the `state/` folder (node keys), ntfy's `data/` folder. [`.gitignore`](.gitignore) excludes the usual paths.
- No open ports on the router, no Quick access / UGREENlink, no [Tailscale Funnel](https://tailscale.com/kb/1223/funnel) for these services.
- Tailscale's default access rules (ACLs) allow every device of a single user to reach every other. Review them before [sharing the NAS](https://tailscale.com/kb/1084/sharing) with someone else.
- Services reached as `http://` are still encrypted on the way, because they travel inside Tailscale.

## Tested on

| | |
|---|---|
| NAS | UGREEN DXP2800 (Intel x86_64), UGOS, Docker app → Project |
| Image | `tailscale/tailscale:stable` (October 2026) |
| Clients | Android, Linux |
| Plan A | ✅ |
| Plan B | ⚠️ untested |
| `/32` route | ⚠️ untested |
| UGREENlink off, UGOS web UI and UGREEN app (Android) through Tailscale | ✅ |
| `watchdog` alerts (container stopped / back up) | ✅ |

Tried it on another UGREEN model (DXP4800, DXP6800, DH series...)? Please open an issue or a PR to add it to this table, especially if you needed plan B.

## Contributing

Corrections and reports from other NAS models are welcome. UGOS menu names change between versions: if what you see on screen differs from this guide, the screen wins, and a PR fixing the guide helps the next person.

## License

[MIT](LICENSE). Not affiliated with UGREEN or Tailscale.
