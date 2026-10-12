# Tailscale sul NAS Ugreen

> **Per l'installazione: [checklist](docs/checklist.md)**, i passi in breve da leggere sul telefono (dettagli nella [guida](docs/guida.md)).

VPN per raggiungere da fuori casa (telefono, PC Windows, Omarchy) i servizi del NAS **senza esporli su internet**: niente porte aperte sul router, niente accesso rapido o UGREENlink di Ugreen.

Nato il 2026-10-12 dal progetto Oura ([`pbi_oura_monitoring`](https://github.com/FabioCuccaro/pbi_oura_monitoring), piano in `docs/piani/vpn.md`), ma tenuto separato perché serve a tutto il NAS: dashboard Oura, interfaccia di UGOS, cartelle condivise, Plex, qBittorrent.

## Com'è fatto

- Un container `tailscale/tailscale` avviato come Project `tailscale` nell'app Docker di UGOS, con la rete dell'host: il NAS entra nella rete Tailscale con il nome `nas` e un indirizzo `100.x.y.z`.
- Sui dispositivi (telefono, PC) c'è l'app Tailscale con lo stesso account.
- Il traffico è cifrato da un capo all'altro (WireGuard). I server di Tailscale coordinano i collegamenti e sanno quali dispositivi si collegano, ma non vedono i dati.
- Modello del compose: [`docker-compose.yml`](docker-compose.yml) (piano A, rete del kernel). Se UGOS non accetta `devices`/`cap_add`: [`docker-compose.userspace.yml`](docker-compose.userspace.yml) (piano B). Guida passo passo: [`docs/guida.md`](docs/guida.md).
- La rete Tailscale (account) esiste già: dentro ci sono `omarchy` (100.98.69.103) e `pixel-8`. Il NAS va aggiunto **a questa**, non a un account nuovo.

## Il NAS

| Cosa | Valore |
|---|---|
| Modello | Ugreen DXP2800, Intel x86_64, 7,5 GB di RAM |
| Indirizzo di casa | `192.168.1.65` (router `192.168.1.254`; prenotazione DHCP da fare) |
| Gestione | Interfaccia web di UGOS, app **Docker** → *Project* (file compose). SSH spento |
| Project già presenti | `oura` (ETL e dashboard Oura, porta 8095, in `docker/oura`), `qbittorrent` (rete host, porta 8888), container `Fabio_Plex` |
| Cartella del Project | `docker/tailscale` nella cartella condivisa `docker` (accesso solo all'amministratore) |

Dettagli completi sul NAS nel repo Oura, `docs/nas.md` punto 6.

## Indirizzi da fuori casa (con l'app Tailscale accesa)

| Servizio | Indirizzo |
|---|---|
| Dashboard Oura | `http://nas:8095` (o `http://100.x.y.z:8095`) |
| Cartella dei dati Oura (Power BI) | `\\192.168.1.65\oura_folder\parquet\` se è attiva la route del passo 6, altrimenti `\\nas\oura_folder\parquet\` |

Da verificare dopo l'installazione e da aggiornare qui (anche l'indirizzo `100.x.y.z` reale).

## Decisioni aperte

| Tema | Stato | Note |
|---|---|---|
| VPN per l'accesso da fuori casa | **Decisa: Tailscale** in un container sul NAS | 2026-10-12. Scelta rispetto a WireGuard sul router (dipende dal router, serve un IP pubblico e una porta aperta) e alla VPN di UGOS (non verificato che esista sul DXP2800). Nessuna porta aperta, app per telefono e Windows |
| Accesso rapido di UGOS (`…ugdocker.link`) e UGREENlink | **Esclusi** per i servizi con dati personali | 2026-10-12. Indirizzo pubblico, protetto solo dalla password del servizio, e il traffico passa dai server Ugreen: il certificato HTTPS è loro, quindi con ogni probabilità lo vedono in chiaro (non verificato). Da spegnere per il container `oura` |
| Repo separato | **Deciso** | Scelta di Fabio, 2026-10-12: Tailscale serve a tutto il NAS, non solo a Oura |
| Nome del nodo | **Deciso**: `nas` | Generico, perché il nodo serve a tutti i servizi |
| Route verso `192.168.1.65` | **Da decidere** dopo il primo test | Utile per Power BI fuori casa senza cambiare il parametro `DataFolder`. Non si annuncia tutta la rete di casa (`192.168.1.0/24`), solo il NAS. 2026-10-12: verificato nel sorgente di Tailscale 1.102.3 (`net/netutil/ip_forward.go`) che per un `/32` che è un indirizzo del NAS stesso **non serve l'inoltro IP**; funziona anche nel piano B. Manca la prova dal vivo |
| Scadenza della chiave del nodo | **Da fare**: *Disable key expiry* sul nodo `nas` | Altrimenti dopo qualche mese il NAS esce dalla rete Tailscale |
| `devices` e `cap_add` nel compose di UGOS | **Da verificare** al primo avvio | Se UGOS non li accetta o manca `/dev/net/tun`: piano B pronto, [`docker-compose.userspace.yml`](docker-compose.userspace.yml). 2026-10-12: Tailscale Serve non serve, perché in userspace le connessioni verso l'IP Tailscale del nodo vengono girate a `127.0.0.1` sulla stessa porta (sorgente v1.102.3, `wgengine/netstack/netstack.go`, `acceptTCP`; non è scritto nella documentazione). Da provare dal telefono, comprese le cartelle condivise |
| Account Tailscale | **Esiste già** | 2026-10-12: su Omarchy `tailscale status` mostra `omarchy` e `pixel-8` nella stessa rete. Il NAS si aggiunge lì (guida, passo 2) |
| Compose controllato in locale | **Fatto**, senza avviarlo | 2026-10-12: `docker compose config` valido per entrambi i file. Non avviati su Omarchy (Docker spento): la prova vera è sul NAS |
| Notifiche sul telefono (ntfy) | **Evolutiva**, dal progetto Oura | Diventa possibile con Tailscale: il telefono raggiunge ntfy sul NAS anche fuori casa |
| Regole di accesso (ACL) | **Default** di Tailscale | Con un solo utente va bene così. Da rivedere se si condivide il NAS con altri |
