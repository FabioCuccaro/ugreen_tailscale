# Guida passo passo (italiano)

Tailscale in un container Docker sul NAS Ugreen, gestito dall'interfaccia web di UGOS: per raggiungere i servizi del NAS da fuori casa senza aprire porte sul router e senza Accesso rapido o UGREENlink. Panoramica e dettagli tecnici nel [README](../../README.md) (in inglese). Versione corta da telefono: [checklist](checklist.md).

I nomi dei menu di UGOS e di Tailscale cambiano con le versioni: se a schermo sono diversi, vale lo schermo (e una PR che corregge la guida è benvenuta).

Nella guida `<IP del NAS>` è l'indirizzo di casa del NAS (per esempio `192.168.1.10`) e `<porta>` è la porta di un servizio del NAS.

## 1. Prima di tutto: spegnere l'accesso rapido

Se in passato hai esposto dei container con l'Accesso rapido di UGOS (`…ugdocker.link`) o UGREENlink:

1. Nell'app **Docker** di UGOS apri il container o il Project e cerca l'opzione dell'accesso rapido (*Quick access* o simile). Disattivala per tutti i servizi con dati personali.
2. Se non la trovi lì: *Pannello di controllo → Device Connection → Remote access* (il nome può variare) e guarda se UGREENlink è attivo e quali container elenca.
3. Prova dal telefono **con il Wi‑Fi spento**: il vecchio indirizzo `…ugdocker.link` non deve più aprirsi.
4. Cambia la password dei servizi che erano esposti: sono stati raggiungibili da internet.

## 2. Account Tailscale (browser)

1. Entra su <https://login.tailscale.com> con l'account che usi già sul telefono e sul PC. **Non** crearne uno nuovo: altrimenti il NAS finisce in una rete diversa e gli altri dispositivi non lo vedono. Se non hai ancora Tailscale, crea l'account ora e installa l'app sul telefono (passo 4).
2. Facoltativo: *Settings → Keys → Generate auth key*, **non** riutilizzabile, **non** effimera. Copiala in un posto sicuro: si vede una volta sola. In alternativa si entra con un link (passo 3).

## 3. Container sul NAS (UGOS)

1. *Docker → Project → Crea*: nome `tailscale`, cartella: nella cartella condivisa `docker` crea `tailscale`.
2. Incolla [`docker-compose.yml`](../../docker-compose.yml) e metti la auth key al posto di `<tailscale auth key>`.
   *Senza auth key:* cancella la riga `TS_AUTHKEY`, avvia il Project, apri i *Logs* del container e apri il link `https://login.tailscale.com/a/…` che compare lì.
3. Avvia. Nei *Logs* non devono esserci errori su `/dev/net/tun` o sui permessi.
4. Su <https://login.tailscale.com/admin/machines> deve comparire `nas` con un indirizzo `100.…`. Annotalo.
5. Sulla riga `nas`, menu `…` → **Disable key expiry**. Altrimenti dopo qualche mese il NAS esce dalla rete Tailscale.
6. Se hai usato la auth key: nel compose togli la riga `TS_AUTHKEY` e fai **Redeploy**. Il nodo deve restare collegato (lo stato è in `docker/tailscale/state`).

Se al passo 3 UGOS rifiuta il compose o i log parlano di `/dev/net/tun`, `devices` o `cap_add`:

1. Annota l'errore esatto.
2. *Project* `tailscale` → modifica il compose: cancella tutto e incolla [`docker-compose.userspace.yml`](../../docker-compose.userspace.yml) (piano B). Se la chiave del primo tentativo è già stata usata, generane una nuova (passo 2).
3. Avvia e riprendi dal punto 4 qui sopra.
4. Il piano B non è ancora stato provato su un NAS vero: se lo usi, apri una issue con il modello del NAS e com'è andata.

## 4. Telefono

1. Installa l'app **Tailscale** e accedi con lo stesso account. Accendila.
2. Spegni il Wi‑Fi e apri `http://nas:<porta>` di un servizio del NAS (un container, l'interfaccia di UGOS): deve rispondere come a casa.
3. Spegni Tailscale nell'app: lo stesso indirizzo non deve rispondere.

Su Android conviene impostare Tailscale come **VPN sempre attiva**: *Impostazioni → Rete e Internet → VPN →* ingranaggio accanto a Tailscale.

## 5. PC (Windows, macOS, Linux)

Installa Tailscale da <https://tailscale.com/download> e accedi con lo stesso account. Da Linux puoi provare il collegamento con `tailscale ping nas`.

## 6. Indirizzi da fuori casa

| Servizio | Indirizzo |
|---|---|
| Un container con una porta pubblicata | `http://nas:<porta>` (o `http://100.x.y.z:<porta>`) |
| Interfaccia di UGOS | `http://nas:<porta di UGOS>`, la stessa che usi a casa |
| Cartelle condivise | `\\nas\<cartella>` da Windows, `smb://nas/<cartella>` da macOS, Linux o dalle app del telefono |

Se il nome `nas` non funziona, controlla che *MagicDNS* sia attivo nella console di Tailscale (pagina *DNS*), oppure usa l'indirizzo `100.x.y.z`.

## 7. Facoltativo: lo stesso indirizzo di casa anche da fuori

Utile se un programma ha salvato l'indirizzo di casa del NAS (per esempio `\\<IP del NAS>\cartella` in un report di Power BI o in un'unità di rete).

1. Sul router fai una **prenotazione DHCP** per l'IP del NAS, così non cambia.
2. Nel compose togli il `#` davanti a `TS_ROUTES`, metti l'IP del NAS (`"<IP del NAS>/32"`) e fai **Redeploy**.
3. Console di Tailscale → *Machines* → `nas` → sezione *Subnets* → **Edit** → spunta la route → **Save**.
4. Windows, iPhone e Android accettano la route da soli. Su Linux serve `sudo tailscale set --accept-routes`.

Annuncia solo il `/32` del NAS, mai tutta la rete di casa (`192.168.1.0/24`). Non serve attivare l'inoltro IP sul NAS: Tailscale lo richiede solo per le route verso altri dispositivi, e salta il controllo quando la route `/32` è un indirizzo della macchina stessa (sorgente di Tailscale, `net/netutil/ip_forward.go`). Non ancora provato dal vivo.

## 8. Facoltativo: notifiche sul telefono

Con [ntfy](ntfy.md) il NAS può mandare notifiche al telefono, sempre solo tramite Tailscale.
