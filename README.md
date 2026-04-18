# Piano Nutrizionale — Web App

App web privata per il piano nutrizionale, tracking peso/misure, ricette, lista spesa e allenamento. Funziona su telefono come app installabile (PWA) e anche offline dopo la prima apertura.

---

## Struttura dei file

```
piano-sonia-site/
├── index.html              ← l'app (tutto in un file)
├── manifest.webmanifest    ← metadata per "Aggiungi alla Home"
├── sw.js                   ← service worker (cache offline)
├── icon.svg                ← icona vettoriale
├── icon-192.png            ← icona Android
├── icon-512.png            ← icona Android (alta risoluzione)
├── icon-180.png            ← icona iOS (apple-touch-icon)
├── .nojekyll               ← disabilita Jekyll su GitHub Pages
├── .gitignore
└── README.md               ← questo file
```

Tutti i dati (peso, misure, ricette fatte, spesa) sono salvati **solo nel browser** del telefono tramite `localStorage`. Non vengono inviati a nessun server.

---

## ⚠️ Privacy — leggi prima di pubblicare

Il file `index.html` contiene:
- dati clinici personali (valori ematici, LDL, ferritina, ecc.)
- nome della persona
- misure antropometriche

Su un **repo GitHub pubblico**, questi dati sono **indicizzati da Google** e chiunque conosca l'URL può leggerli. Il meta tag `noindex` scoraggia i motori di ricerca ma non impedisce l'accesso diretto.

**Raccomandazione**: usa **repo privato** + un servizio di hosting che accetta repo privati gratuitamente (Netlify, Vercel o Cloudflare Pages). Vedi Opzione C sotto.

---

## Opzione A — GitHub Pages (repo pubblico, gratis) ⚠️

**Pro**: tutto su GitHub, zero servizi aggiuntivi, gratis.
**Contro**: repo pubblico → dati clinici leggibili a chiunque conosca l'URL.

### Passaggi

1. Crea un nuovo repository pubblico su GitHub, es. `piano-nutrizionale`.
2. Da terminale, dentro questa cartella:
   ```bash
   git init
   git add .
   git commit -m "Initial commit: piano nutrizionale PWA"
   git branch -M main
   git remote add origin https://github.com/<TUO-USERNAME>/piano-nutrizionale.git
   git push -u origin main
   ```
3. Su GitHub: **Settings → Pages**.
   - Source: `Deploy from a branch`
   - Branch: `main`, folder: `/ (root)`
   - Save.
4. Attendi ~1 minuto. L'URL sarà `https://<TUO-USERNAME>.github.io/piano-nutrizionale/`.

---

## Opzione B — GitHub Pages (repo privato, richiede GitHub Pro)

GitHub Pages supporta repo privati **solo con piano Pro o superiore** (~4 $/mese).

Stessi passaggi dell'Opzione A, ma crea il repository come **Private** e assicurati di avere un piano GitHub Pro attivo.

---

## Opzione C — Netlify / Vercel / Cloudflare Pages (repo privato + gratis) ✅ CONSIGLIATO

Hosting gratuito anche da repo GitHub **privati**. Setup in 3 minuti.

### C.1 — Netlify (più semplice)

1. Crea repository **privato** su GitHub e fai push (stessi comandi dell'Opzione A).
2. Vai su [netlify.com](https://netlify.com) → Sign up con GitHub.
3. `Add new site` → `Import an existing project` → GitHub → seleziona il repo.
4. Build settings: lascia tutto vuoto (è un sito statico).
5. `Deploy site`. URL tipo: `https://random-name.netlify.app`.
6. (Opzionale) in **Site settings → Change site name** metti es. `piano-sonia` → `piano-sonia.netlify.app`.
7. (Opzionale, consigliato) in **Site settings → Access control → Password protection**: proteggi il sito con una password. Costa 0€ su piano free solo come Basic Auth a livello di build — altrimenti serve piano Pro. In alternativa puoi usare Cloudflare Access (C.3).

### C.2 — Vercel

1. Push su GitHub privato.
2. [vercel.com](https://vercel.com) → Sign up con GitHub.
3. `Add New → Project` → importa il repo → `Deploy` (nessuna configurazione necessaria).
4. URL: `https://piano-xxx.vercel.app`.

### C.3 — Cloudflare Pages (il più privacy-friendly)

1. Push su GitHub privato.
2. [dash.cloudflare.com](https://dash.cloudflare.com) → Workers & Pages → `Create application` → `Pages` → `Connect to Git`.
3. Seleziona il repo, `Save and Deploy`.
4. URL: `https://piano-xxx.pages.dev`.
5. **Bonus**: attiva **Cloudflare Access** (gratis fino a 50 utenti) → protegge il sito con login via email / Google. Così solo le email autorizzate possono aprirlo.

---

## Come installare l'app sul telefono

Una volta che il sito è online:

### Su iPhone (Safari)
1. Apri l'URL in **Safari**.
2. Tocca il pulsante condividi (quadrato con freccia in su).
3. Scorri → `Aggiungi alla schermata Home`.
4. L'icona compare come una normale app. Aprendola, parte a tutto schermo senza barra del browser.

### Su Android (Chrome)
1. Apri l'URL in **Chrome**.
2. Menu (⋮) → `Installa app` (o `Aggiungi a schermata Home`).
3. Icona sulla home, apertura in finestra dedicata.

Dopo la prima apertura, l'app funziona **anche senza connessione** (grazie al service worker).

---

## Aggiornare il sito dopo modifiche

Modifica i file, poi:
```bash
git add .
git commit -m "Aggiornamento piano / ricette / ..."
git push
```
Il sito si aggiorna automaticamente in 1-2 minuti. Se hai già aperto la PWA sul telefono, per forzare il reload della cache chiudi e riapri l'app (o aggiorna la versione in `sw.js` — cambia `piano-nutrizionale-v1` → `v2`).

---

## Note tecniche

- **Single-file app**: tutto il codice JS/CSS/HTML è dentro `index.html`. Niente build step, niente npm.
- **Chart.js** viene caricato da CDN (cdnjs) alla prima apertura e poi messo in cache dal service worker.
- **Persistenza dati**: `localStorage` sotto chiave `sonia_nutrition_v1`. I dati restano sul telefono anche dopo la chiusura. Per backup: nella tab "Dashboard" puoi esportare JSON (se non c'è ancora il bottone, basta eseguire in console `copy(localStorage.getItem('sonia_nutrition_v1'))`).
- **Nessuna analytics, nessun tracking esterno**. L'unica chiamata esterna è il CDN Chart.js.

---

## Troubleshooting

- **"Il service worker non si registra"** → serve HTTPS. Funziona su `localhost` ma non su file:// né http://. GitHub Pages / Netlify / Vercel / Cloudflare danno tutti HTTPS di default.
- **"Su iPhone l'app non si aggiorna"** → la PWA di iOS è aggressiva col cache. Elimina l'app dalla home e reinstallala, oppure incrementa la versione in `sw.js`.
- **"Google mi ha indicizzato il sito pubblico"** → hai scelto Opzione A. Se vuoi riservatezza, migra a Opzione C (Cloudflare Access). Il meta `noindex` aiuta ma non è garantito.
