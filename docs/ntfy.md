# ntfy: push notifications from the NAS (optional)

[ntfy](https://ntfy.sh) is a small notification server. It runs on the NAS as the Project `ntfy`, and your phone reaches it **only through Tailscale** at `http://nas:8090`: no open ports, no Quick access, no external push service (Firebase included).

Anything on the NAS (a script, a container, a monitoring tool such as Uptime Kuma) can send a message to a *topic*, for example `nas`, and the phone subscribed to that topic shows it as a notification.

Compose: [`docker/ntfy/docker-compose.yml`](../docker/ntfy/docker-compose.yml). It contains no password: the user is created in step 3 and lives only on the NAS.

Italian version: [`it/ntfy.md`](it/ntfy.md).

## 1. Check that port 8090 is free

UGOS → **Docker** → *Container*: look at the ports already in use. If **8090** is taken, replace `8090` in **two** places in the compose: `ports` and `NTFY_BASE_URL`.

## 2. Create the `ntfy` Project

1. Docker → **Project** → *Create*. Name `ntfy`, folder `docker/ntfy` (inside the `docker` shared folder, next to `tailscale`).
2. Paste [`docker/ntfy/docker-compose.yml`](../docker/ntfy/docker-compose.yml). Change `TZ` to your time zone, and the port if needed.
3. Start it. The container Logs must show a line like `Listening on :80[http], ntfy 2.30.0`.
4. From a computer at home open `http://<NAS LAN IP>:8090`: the ntfy page opens, but you can't do anything yet. That's intended: nobody is authorized.

## 3. Create your user

Once, from a terminal **inside the container** (no SSH needed).

1. Docker → *Container* → `ntfy` → container terminal → **Add**. In *Add Connection*, set *Command* to **`/bin/sh`** instead of `/bin/bash` (the ntfy image is Alpine and has no bash) → *Confirm*.
2. Run (replace `<user>` with the name you want):

   ```
   ntfy user add --role=admin <user>
   ```

3. Type a new password twice (nothing is shown while typing). It answers `user <user> added with role admin`.
4. Save the password in your password manager.

## 4. Test from a computer

1. Open `http://<NAS LAN IP>:8090` → *Sign in* with your user.
2. *Subscribe to topic* → `nas`.
3. Send a test message from the topic page: it appears in the list.

## 5. The phone (Android)

**Tailscale always on.** Notifications arrive only while the phone is on your tailnet: Android *Settings* → *Network & internet* → *VPN* → gear next to **Tailscale** → enable **Always-on VPN**.

**ntfy app.**

1. Install **ntfy** from the Play Store (or F-Droid).
2. *Settings* → **Default server**: `http://nas:8090`, so new topics use your NAS instead of `ntfy.sh`.
3. *Settings* → **Manage users** → *Add user*: server `http://nas:8090`, your user and password.
4. Main screen → **+** → topic `nas`. Keep **Instant delivery** on if offered → *Subscribe*.
5. A permanent notification appears ("Listening for incoming notifications"): it is the service keeping the connection open, and it is needed. You can hide that notification category; the service keeps running.
6. Android *Settings* → *Apps* → *ntfy* → *App battery usage* → **Unrestricted**, otherwise messages may be delayed when the phone is idle.

### Why it works without Firebase

With the public `ntfy.sh` server, the Android app uses Google's Firebase. With your own server it keeps a direct, always-open connection instead ("instant delivery"), which is why Tailscale must stay on and the app needs unrestricted battery ([ntfy docs](https://docs.ntfy.sh/subscribe/phone/#instant-delivery)). If the phone is offline for a while, messages wait on the NAS for **72 hours**.

iOS works differently (it relies on Apple push through `ntfy.sh` for instant delivery); see the [ntfy docs](https://docs.ntfy.sh/subscribe/phone/) before relying on it.

## 6. Real test, away from home

1. Turn Wi‑Fi off on the phone (Tailscale on).
2. From the ntfy web page, send a message to the `nas` topic.
3. The notification arrives within seconds, also with the phone locked.

## Sending from scripts and containers

Use a **token** instead of the password. In the container terminal (step 3):

```
ntfy token add <user>
```

It returns a `tk_…` code. It is a secret like a password: put it only in the compose on the NAS, never in git. From the NAS itself or other containers use the NAS LAN address (the name `nas` only resolves on devices running Tailscale):

```
curl -H "Authorization: Bearer tk_…" -H "Title: Backup" -d "Backup finished" http://<NAS LAN IP>:8090/nas
```

## Security notes

- Without login nothing can be read or written (`NTFY_AUTH_DEFAULT_ACCESS: deny-all`): an anonymous publish gets `403`.
- The connection is `http`, not `https`. Away from home it travels inside Tailscale, which encrypts it. At home the password crosses only your LAN: acceptable as long as port 8090 is never opened on the router.
- Do **not** put the `ntfy` Project on Quick access or UGREENlink.
