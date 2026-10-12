# Regole per gli assistenti (Claude Code, thread sul web, altri agenti)

Questo repo raccoglie la configurazione di **Tailscale sul NAS Ugreen di casa**: la VPN che permette di raggiungere i servizi del NAS da fuori casa senza esporli su internet. Si lavora da più PC e da più sessioni che non condividono la cronologia delle chat: **il README è la memoria comune**.

1. **README sempre aggiornato, nello stesso commit della modifica**, compresa la tabella "Decisioni aperte" (stato, data, motivo). Se una decisione cambia, aggiorna la riga invece di aggiungerne una nuova.
2. **Mai segreti in git**: auth key di Tailscale, cartella `state/`, compose compilato, password. Nel repo solo il modello con i valori tra `< >`.
3. **Niente esposizione su internet.** Niente accesso rapido o UGREENlink per i servizi del NAS, niente porte aperte sul router, niente Tailscale Funnel. Fuori casa si passa solo dalla rete Tailscale.
4. **Verifica sulla documentazione ufficiale** (tailscale.com, documentazione Ugreen) prima di scrivere variabili, opzioni o menu. Se non puoi verificare, scrivilo e indica a Fabio cosa controllare a schermo.
5. **Il NAS si gestisce dall'interfaccia web di UGOS.** SSH è spento: gli assistenti non hanno accesso diretto al NAS. I passi su UGOS, sulla console di Tailscale e sul telefono sono manuali per Fabio, scritti passo passo (è poco pratico di terminale).
6. Testo per l'utente in italiano semplice.
7. I dati del progetto Oura (`FabioCuccaro/pbi_oura_monitoring`) sono dati sanitari: qui non entra niente di quei dati. Quel repo rimanda qui per Tailscale; se cambia l'indirizzo con cui si raggiungono i suoi servizi, aggiorna anche il suo README.
