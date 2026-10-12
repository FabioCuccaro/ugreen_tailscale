# Checklist: Tailscale sul NAS

Versione corta della [guida](guida.md), da leggere sul telefono (app GitHub o browser, con il tuo account: il repo è privato). Per i dettagli di ogni punto apri la guida al passo indicato.

Dove si fa:
- 🏠 **a casa**: serve la rete di casa, perché UGOS si apre solo da lì finché Tailscale non funziona.
- 🌍 **ovunque**: basta internet.
- 🚶 **fuori casa**: è la prova vera.

## Prima: ovunque 🌍

- [ ] **Auth key** (guida, passo 2). Apri <https://login.tailscale.com/admin/settings/keys> con l'account che usi già su Omarchy e sul Pixel 8 (**non** crearne uno nuovo) → *Generate auth key*: non riutilizzabile, non effimera. Copiala in un posto sicuro (non in una chat né in questo repo): si vede una volta sola.

## A casa, dal PC 🏠

- [ ] **Spegni l'accesso rapido** (guida, passo 1). App Docker di UGOS → container `oura` → opzione *Quick access* o simile: disattivala. Dal telefono, **con il Wi‑Fi spento**, il vecchio indirizzo `…ugdocker.link` non deve più aprirsi.
- [ ] **Cambia la password della dashboard Oura** (`OURA_DASHBOARD_PASSWORD` nel Project `oura` → *Redeploy*).
- [ ] **Crea il Project `tailscale`** (guida, passo 3). Docker → Project → Crea, nome `tailscale`, cartella `docker/tailscale`. Incolla [`docker-compose.yml`](../docker-compose.yml) e metti la auth key al posto di `<auth key di Tailscale>`. Avvia.
  - Se UGOS rifiuta il testo o i *Logs* parlano di `/dev/net/tun`, `devices` o `cap_add`: **fai una foto all'errore**, poi incolla [`docker-compose.userspace.yml`](../docker-compose.userspace.yml) (piano B) e riavvia.
- [ ] **Controlla il nodo**. Su <https://login.tailscale.com/admin/machines> deve comparire `nas` con un indirizzo `100.…`. **Annotalo.**
- [ ] Sulla riga `nas`: menu `…` → **Disable key expiry**.
- [ ] **Togli la chiave dal compose**: cancella la riga `TS_AUTHKEY` → *Redeploy*. `nas` deve restare collegato nella console.

## Prova dal telefono, ancora a casa 🏠

- [ ] App Tailscale sul Pixel 8: accendila.
- [ ] **Spegni il Wi‑Fi** e apri <http://nas:8095>: deve chiedere utente e password e mostrare la dashboard Oura.
- [ ] Spegni Tailscale nell'app: lo stesso indirizzo **non** deve rispondere. Poi riaccendila.

Se fin qui funziona, il grosso è fatto: da ora anche UGOS dovrebbe aprirsi da fuori, all'indirizzo `100.…` annotato con la stessa porta che usi a casa (da provare).

## Fuori casa 🚶

- [ ] Telefono con Tailscale acceso, su rete mobile o Wi‑Fi di altri: <http://nas:8095> si apre.
- [ ] Se hai il PC Windows con te: installa Tailscale da <https://tailscale.com/download>, stesso account, e apri <http://nas:8095>.
- [ ] Facoltativo, solo se vuoi Power BI fuori casa (guida, passo 6). Nel Project `tailscale` togli il `#` davanti a `TS_ROUTES` → *Redeploy* → console → `nas` → *Subnets* → *Edit* → spunta `192.168.1.65/32` → *Save*. Poi da Windows apri `\\192.168.1.65\oura_folder`.

## Cosa mandarmi dopo

- L'indirizzo `100.…` di `nas`.
- Piano A o piano B, e la foto dell'errore se c'è stato.
- Quali prove sono riuscite e quali no.
- Dove si trovava l'opzione dell'accesso rapido in UGOS.

Con questi dati aggiorno il README di questo repo e quello del repo Oura.
