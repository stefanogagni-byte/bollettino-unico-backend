# Bollettino Unico — Backend notifiche

Monitora i bollettini di viabilità e invia una notifica push quando
qualcosa cambia su una tratta che l'utente segue.

## Come funziona

```
lib/source.js   -> recupera i bollettini (per ora dati di esempio,
                    va collegato al feed reale CCISS quando avrai la convenzione)
lib/store.js    -> salva iscrizioni e l'ultimo stato noto (file JSON, sostituibile con un DB vero)
lib/push.js     -> invia le notifiche push (Web Push standard, funziona su Chrome/Edge/Firefox/Android; su iOS serve la PWA installata sulla home)
lib/poller.js   -> ogni N minuti confronta i nuovi bollettini con i precedenti e, se cambia qualcosa, notifica chi segue quella tratta
server.js       -> API + avvio del poller
public/         -> pagina minima per attivare le notifiche dal browser
```

## Setup (5 minuti)

1. Installa le dipendenze:
   ```
   npm install
   ```

2. Genera le chiavi VAPID (servono per autenticare le notifiche push, sono gratuite e locali):
   ```
   npx web-push generate-vapid-keys
   ```
   Copia `.env.example` in `.env` e incolla le due chiavi generate:
   ```
   cp .env.example .env
   ```

3. Avvia il server:
   ```
   npm start
   ```

4. Apri `http://localhost:3000`, inserisci una tratta (es. "A4") e attiva le notifiche.

5. Per testare subito senza aspettare il primo intervallo:
   ```
   curl -X POST http://localhost:3000/api/forza-controllo
   ```
   Se cambi manualmente qualcosa in `lib/source.js` (es. aggiungi un bollettino nuovo
   o cambi il `level` di uno esistente) e rifai la chiamata sopra, dovresti ricevere
   la notifica sul browser in cui hai attivato l'iscrizione.

## Collegare i dati reali (CCISS)

In `lib/source.js`, la funzione `fetchBollettini()` è l'unico punto da
modificare: sostituisci i dati di esempio con la chiamata vera al feed
CCISS/Viabilità Italia una volta ottenuta la convenzione d'uso, mantenendo
la stessa forma dei dati (`id`, `title`, `level`, `start`, `end`, `tratte`, `desc`).
Tutto il resto — storage, confronto, invio notifiche — funziona già così com'è.

## Deploy (per farlo girare 24/7, non solo sul tuo computer)

Il poller deve girare in continuazione per controllare a intervalli regolari,
quindi serve un hosting che tenga il processo Node.js sempre acceso. Opzioni
economiche/gratuite per iniziare: Railway, Render, Fly.io. Tutte supportano
un `npm start` diretto da un repository Git, senza configurazione server aggiuntiva.

Nota importante: lo storage a file JSON (`data/`) va bene per prototipare,
ma su molti hosting il filesystem non è persistente tra i riavvii — per la
versione con utenti reali conviene passare a un database (anche gratuito,
es. Postgres su Railway/Supabase), cambiando solo il contenuto di `lib/store.js`.

## Limiti da conoscere

- Le notifiche push web funzionano su Android e desktop. Su iPhone
  funzionano solo se la pagina viene "aggiunta alla Home" come app (PWA) —
  Safari non supporta le push per pagine aperte nel browser normale.
- Questo è un MVP a singolo processo: va benissimo per pochi/centinaia di
  iscritti. Con volumi più alti servirebbe una coda di invio e un database vero.
