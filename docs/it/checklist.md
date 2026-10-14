# Checklist: Tailscale sul NAS

Versione corta della [guida](guida.md), da leggere sul telefono. Per i dettagli di ogni punto apri la guida al passo indicato.

Dove si fa:
- 🏠 **a casa**: serve la rete di casa, perché UGOS si apre solo da lì finché Tailscale non funziona.
- 🌍 **ovunque**: basta internet.
- 🚶 **fuori casa**: è la prova vera.

## Prima: ovunque 🌍

- [ ] **Account** (guida, passo 2). Usa l'account Tailscale che hai già sul telefono e sul PC, **non** crearne uno nuovo.
- [ ] Facoltativo: **auth key**. <https://login.tailscale.com/admin/settings/keys> → *Generate auth key*: non riutilizzabile, non effimera. Copiala in un posto sicuro (non in una chat né in un repo): si vede una volta sola.

## A casa, dal PC 🏠

- [ ] **Spegni UGREENlink** (guida, passo 1): *Control Panel → Remote access → Method 1* → togli la spunta a *UGREENlink remote access*. Spegne anche l'accesso rapido dei container (non c'è un interruttore per container). Dal telefono, **con Wi‑Fi e Tailscale spenti**, `ug.link/<ID>` e `…ugdocker.link` non devono più aprirsi. Cambia la password dei servizi che erano esposti. Da fuori casa UGOS si apre poi su `https://nas:9443`.
- [ ] **Crea il Project `tailscale`** (guida, passo 3). Docker → Project → Crea, nome `tailscale`, cartella `docker/tailscale`. Incolla [`docker-compose.yml`](../../docker-compose.yml) e metti la auth key al posto di `<tailscale auth key>` (oppure togli la riga e usa il link di login che compare nei *Logs*). Avvia.
  - Se UGOS rifiuta il testo o i *Logs* parlano di `/dev/net/tun`, `devices` o `cap_add`: **fai una foto all'errore**, poi incolla [`docker-compose.userspace.yml`](../../docker-compose.userspace.yml) (piano B) e riavvia.
- [ ] **Controlla il nodo**. Su <https://login.tailscale.com/admin/machines> deve comparire `nas` con un indirizzo `100.…`. **Annotalo.**
- [ ] Sulla riga `nas`: menu `…` → **Disable key expiry**.
- [ ] Se hai usato la auth key, **toglila dal compose**: cancella la riga `TS_AUTHKEY` → *Redeploy*. `nas` deve restare collegato nella console.

## Prova dal telefono, ancora a casa 🏠

- [ ] App Tailscale sul telefono: accendila.
- [ ] **Spegni il Wi‑Fi** e apri `http://nas:<porta>` di un servizio del NAS: deve rispondere.
- [ ] Spegni Tailscale nell'app: lo stesso indirizzo **non** deve rispondere. Poi riaccendila.

## Fuori casa 🚶

- [ ] Telefono con Tailscale acceso, su rete mobile o Wi‑Fi di altri: `http://nas:<porta>` si apre.
- [ ] Cartelle condivise: `\\nas\<cartella>` da Windows o un'app SMB sul telefono.
- [ ] Facoltativo (guida, passo 7): route `/32` verso l'IP di casa del NAS.

Se qualcosa non va come scritto qui, apri una issue sul repo con il modello del NAS e la versione di UGOS.
