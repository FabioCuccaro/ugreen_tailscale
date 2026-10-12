# Guida passo passo

Per Fabio. I nomi dei menu di UGOS e di Tailscale vengono dalle guide trovate il 2026-10-12: se a schermo sono diversi, vale lo schermo, e questa guida si corregge.

## 1. Prima di tutto: spegnere l'accesso rapido

1. Nell'app **Docker** di UGOS apri il container o il Project `oura` e cerca l'opzione dell'accesso rapido (*Quick access* o simile). Disattivala. Fai lo stesso per gli altri container che la usano.
2. Se non la trovi lì: *Pannello di controllo → Device Connection → Remote access* (nome da verificare) e guarda se UGREENlink è attivo e se elenca i container.
3. Prova dal telefono **con il Wi‑Fi spento**: il vecchio indirizzo `…ugdocker.link` non deve più aprirsi.
4. Cambia la password della dashboard Oura (`OURA_DASHBOARD_PASSWORD` nel compose del Project `oura`, poi **Redeploy**): è stata raggiungibile da internet.
5. Annota qui e nel repo Oura (`docs/nas.md`, punto 6) dove sta l'opzione.

## 2. Account Tailscale (browser)

1. Crea l'account su <https://login.tailscale.com> (piano *Personal*, gratuito), per esempio con l'account Google o GitHub.
2. *Settings → Keys → Generate auth key*: **non** riutilizzabile, **non** effimera. Copiala: si vede una volta sola.

## 3. Container sul NAS (UGOS)

1. *Docker → Project → Crea*: nome `tailscale`, cartella: nella cartella condivisa `docker` crea `tailscale` (come per `oura`).
2. Incolla [`docker-compose.yml`](../docker-compose.yml) e metti la auth key al posto di `<auth key di Tailscale>`.
3. Avvia. Nei *Logs* non devono esserci errori su `/dev/net/tun` o sui permessi.
4. Su <https://login.tailscale.com/admin/machines> deve comparire `nas` con un indirizzo `100.…`. Annotalo nel README.
5. Sulla riga `nas`, menu `…` → **Disable key expiry**.
6. Nel compose togli la riga `TS_AUTHKEY` e fai **Redeploy**: il nodo deve restare collegato (lo stato è in `docker/tailscale/state`).

Se al passo 3 UGOS rifiuta il compose o i log parlano di `/dev/net/tun`: annota l'errore esatto e chiedi, la strada alternativa è la modalità userspace.

## 4. Telefono

1. Installa l'app **Tailscale** e accedi con lo stesso account.
2. Spegni il Wi‑Fi e apri `http://nas:8095`. Deve chiedere utente e password e mostrare la dashboard Oura.
3. Spegni Tailscale nell'app: lo stesso indirizzo non deve rispondere.

## 5. PC Windows e Omarchy (se servono da fuori)

- **Windows**: installa Tailscale da <https://tailscale.com/download>, accedi con lo stesso account.
- **Omarchy**: installa il pacchetto `tailscale`, poi `sudo systemctl enable --now tailscaled` e `sudo tailscale up`. Da verificare sulla documentazione di Arch prima di farlo.

## 6. Facoltativo: Power BI fuori casa con lo stesso indirizzo

Power BI legge `\\192.168.1.65\oura_folder\parquet\`. Per non cambiare il parametro `DataFolder` fuori casa:

1. Nel compose togli il `#` davanti a `TS_ROUTES: "192.168.1.65/32"` e fai **Redeploy**.
2. Console di Tailscale → *Machines* → `nas` → sezione *Subnets* → **Edit** → spunta `192.168.1.65/32` → **Save**.
3. Windows, iPhone e Android accettano la route da soli. Su Omarchy (Linux) serve `sudo tailscale set --accept-routes`.
4. Fuori casa, con Tailscale acceso, aggiorna il report in Power BI.

Da verificare: che annunciare l'indirizzo del NAS dal NAS stesso funzioni senza attivare l'inoltro IP (dovrebbe, perché i pacchetti sono destinati al NAS e non vanno inoltrati altrove).

## 7. Dopo

Aggiorna il README (indirizzi reali, righe "Da verificare" e "Da fare" in *Decisioni aperte*) e la riga "Accesso ai dati da fuori casa" del README del repo Oura.
