# Rules for AI assistants and contributors

This repo is a public, generic guide: Tailscale in Docker on a UGREEN NAS managed from the UGOS web UI, to reach the NAS's services from outside without exposing them to the internet.

1. **Keep it generic.** No personal values: real IP addresses, Tailscale `100.x.y.z` addresses, device names, user names, personal services or paths. Use placeholders between `< >` (`<NAS LAN IP>`, `<port>`, `<user>`) or clearly fake examples (`192.168.1.10`).
2. **Never commit secrets**: Tailscale auth keys, the `state/` folder, filled-in compose files, passwords, ntfy tokens.
3. **No internet exposure.** No UGOS Quick access or UGREENlink, no open router ports, no Tailscale Funnel. From outside, only through the tailnet.
4. **Check the official docs** (tailscale.com, docs.ntfy.sh, UGREEN) before writing variables, options or menu names. Mark anything not verified on a real NAS as untested (⚠️ in the README).
5. **Readers manage the NAS from the UGOS web UI**, often without SSH and with little terminal experience: write UGOS, Tailscale console and phone steps one by one.
6. **English README is the entry point**, the Italian guides live in `docs/it/`. When a step changes, update both languages in the same commit.
7. When someone reports a result on a new model or setup, update the "Tested on" table in the README.
