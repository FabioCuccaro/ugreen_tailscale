# ntfy: notifiche sul telefono dal NAS (facoltativo)

Versione inglese: [`../ntfy.md`](../ntfy.md).

ntfy è un piccolo server di notifiche. Gira sul NAS come Project `ntfy`, e il telefono lo raggiunge **solo tramite Tailscale** all'indirizzo `http://nas:8090`: nessuna porta aperta sul router, niente Accesso rapido, niente server esterni (Firebase compreso).

Qualsiasi cosa sul NAS (uno script, un container, uno strumento di monitoraggio come Uptime Kuma) può mandare un messaggio a un "argomento" (*topic*), per esempio `nas`, e il telefono iscritto a quell'argomento lo riceve come notifica.

Modello del compose: [`docker/ntfy/docker-compose.yml`](../../docker/ntfy/docker-compose.yml). Non contiene password: l'utente si crea al passo 3 e resta solo sul NAS.

Dove si fa: 🏠 a casa, dal PC · 📱 sul telefono.

## 1. Controlla che la porta 8090 sia libera 🏠

UGOS → app **Docker** → *Container*: guarda le porte già usate dai container. Se qualcuno usa già **8090**, nel compose sostituisci `8090` con un'altra porta (per esempio `8091`) in **due** punti: `ports` e `NTFY_BASE_URL`.

## 2. Crea il Project `ntfy` 🏠

1. Docker → **Project** → *Crea*. Nome `ntfy`, cartella `docker/ntfy` (nella cartella condivisa `docker`, come `tailscale`).
2. Incolla il contenuto di [`docker/ntfy/docker-compose.yml`](../../docker/ntfy/docker-compose.yml). Cambia `TZ` se non sei nel fuso di Roma, e la porta se al passo 1 era occupata.
3. Avvia. Nei *Logs* del container `ntfy` deve comparire una riga come `Listening on :80[http], ntfy 2.30.0`.
4. Dal PC di casa apri <http://<IP del NAS>:8090>: deve aprirsi la pagina di ntfy. Per ora non puoi fare niente, perché nessuno è ancora autorizzato: è voluto.

## 3. Crea il tuo utente 🏠

Va fatto una volta sola, da un terminale **dentro il container** (non serve SSH).

1. Docker → *Container* → `ntfy` → terminale del container → **Add**. Nella finestra *Add Connection*, nel campo *Command* scrivi **`/bin/sh`** al posto di `/bin/bash` (il container di ntfy non ha bash) → *Confirm*.
2. Scrivi questo e premi Invio:

   ```
   ntfy user add --role=admin <utente>
   ```

3. Scrivi due volte una password nuova (mentre scrivi non si vede niente, è normale). Deve rispondere `user <utente> added with role admin`.
4. Annota la password nel tuo password manager.

Se il terminale si chiude subito o dà errore, controlla che il comando sia `/bin/sh` e che il container `ntfy` sia avviato.

## 4. Prova dal PC 🏠

1. Apri <http://<IP del NAS>:8090> → *Sign in* (o *Accedi*) → il tuo utente e la password.
2. *Subscribe to topic* (o *Iscriviti*), nome `nas`.
3. Nella pagina dell'argomento scrivi un messaggio di prova e invialo: deve comparire nella lista.

## 5. Il telefono 📱

Questa parte è per Android.

**Tailscale sempre acceso.** Le notifiche arrivano solo se il telefono è collegato alla rete Tailscale. Così non si spegne da solo: *Impostazioni* di Android → *Rete e Internet* → *VPN* → ingranaggio accanto a **Tailscale** → attiva **VPN sempre attiva**.

**App ntfy.**

1. Installa **ntfy** dal Play Store (sviluppatore: Philipp C. Heckel), oppure da F-Droid.
2. Nell'app: *Settings* → **Default server**: `http://nas:8090`. Così ogni argomento nuovo usa il NAS e non `ntfy.sh`.
3. *Settings* → **Manage users** → *Add user*: server `http://nas:8090`, il tuo utente e la password del passo 3.
4. Torna alla schermata principale → **+** → argomento `nas`. Se compare la spunta **Instant delivery**, lasciala attiva. → *Subscribe*.
5. Comparirà una notifica fissa ("Listening for incoming notifications" o simile): è il servizio che tiene aperta la connessione col NAS, e serve. Se ti dà fastidio, tienila premuta e disattiva solo quella categoria di notifiche: il servizio continua a funzionare.
6. **Batteria senza limiti per ntfy.** *Impostazioni* di Android → *App* → *ntfy* → *Utilizzo batteria dell'app* → **Senza limitazioni**. Altrimenti Android, quando il telefono è fermo, può addormentare l'app e i messaggi arrivano in ritardo.

I nomi esatti dei menu dell'app ntfy possono cambiare con le versioni: se uno non c'è, cerca l'opzione con un nome simile.

### Perché funziona senza Firebase

Le app Android di solito ricevono le notifiche tramite Firebase, il servizio di Google. L'app ntfy usa Firebase **solo** per il server pubblico `ntfy.sh`; con un server proprio come il NAS tiene invece una connessione diretta e sempre aperta ("instant delivery"), con quella notifica fissa. Per questo le notifiche arrivano in pochi secondi, anche a telefono fermo, ma servono Tailscale acceso e l'app senza limiti di batteria (fonte: [documentazione ntfy, Instant delivery](https://docs.ntfy.sh/subscribe/phone/#instant-delivery)).

Se il telefono resta scollegato per un po' (Tailscale spento, niente rete), i messaggi restano sul NAS per **72 ore** e arrivano appena si ricollega.

## 6. Prova vera, fuori casa 📱

1. Spegni il Wi‑Fi del telefono (Tailscale acceso).
2. Dal PC di casa, nella pagina di ntfy, manda un messaggio all'argomento `nas`.
3. Sul telefono deve arrivare la notifica entro pochi secondi.
4. Prova anche a telefono bloccato e fermo per qualche minuto.

## Per gli script e gli altri container (dopo)

Per mandare notifiche da uno script o da un altro container serve un **token** al posto della password. Nel terminale del container `ntfy` (passo 3):

```
ntfy token add <utente>
```

Risponde con un codice `tk_…`: è un segreto come la password, va solo nel compose sul NAS, mai in git. Esempio di invio dal NAS (dagli altri container si usa l'indirizzo di casa del NAS, non `nas`, che funziona solo sui dispositivi con l'app Tailscale):

```
curl -H "Authorization: Bearer tk_…" -H "Title: Backup" -d "Backup finito" http://<IP del NAS>:8090/nas
```

## Note di sicurezza

- Senza login non si legge e non si scrive niente (`NTFY_AUTH_DEFAULT_ACCESS: deny-all`): un invio senza credenziali riceve `403` (provato su ntfy 2.30.0).
- Il collegamento è `http`, non `https`. Fuori casa passa dentro Tailscale, che lo cifra. A casa la password viaggia in chiaro solo sulla rete di casa: accettabile, a patto di non aprire mai la porta 8090 sul router.
- Il Project `ntfy` **non** va messo su Accesso rapido o UGREENlink.
