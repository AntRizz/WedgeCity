# Wedge City – Specifica della Mappa

> **Come è stata applicata (27/11/2026)** – scostamenti voluti dalla specifica originale qui sotto:
> - **Tile:** CARTO Dark Matter ora chiede una chiave a pagamento (schermata "API KEY REQUIRED", issue #1).
>   Al suo posto: **OpenFreeMap**, stile `dark` (https://tiles.openfreemap.org/styles/dark), gratis e senza chiave,
>   disegnato dentro Leaflet con `@maplibre/maplibre-gl-leaflet` 0.1.4 + `maplibre-gl` 4.7.1, ricolorato con un filtro CSS.
>   Niente `tile.openstreetmap.org`.
> - **Coordinate:** in `data/luoghi.json` Merlo, Lidl e cassonetto abiti hanno `lat`/`lon` a `null` (non verificate):
>   non vengono mostrati e compaiono in "Luoghi senza coordinate" con il pulsante **Posiziona sulla mappa**;
>   la posizione scelta si salva sul telefono (`coord`) e va nel backup.
> - **Casa** non sta nel JSON perché il repository è pubblico: si aggiunge con la pressione lunga (resta solo sul telefono).
> - I luoghi dell'utente stanno nello stato dell'app (`luoghi`, chiave `m15`) e quindi nel backup, non in `wc_luoghi`.
> - Le missioni personali con un luogo compaiono nel popup ("Missioni qui"); `missione` nel JSON apre la scheda indicata (es. `spesa`).

---


Documento di passaggio per la sessione che sviluppa **Wedge City** (repo `AntRizz/WedgeCity`, GitHub Pages).
Descrive *cosa* costruire e *come*, con un approccio ispirato al sito "La Scuola Cerca" (mappa Leaflet su sito statico).

> ⚠️ Il codice di "La Scuola Cerca" è "tutti i diritti riservati": **non copiarlo**. Qui c'è solo l'approccio, e il codice di esempio sotto è scritto da zero.

---

## 1. Obiettivo

Una scheda **Mappa** in stile GTA (scura, neon, icone a "blip") che mostra i luoghi della vita di Antonio a Cuneo (Lidl, casa, lavoro, palestra, ecc.), la sua posizione GPS e, per ogni luogo, i pulsanti **Naviga con Google Maps** e **Naviga con Waze**.

## 2. Principi (le idee da riprendere)

1. **Sito statico, zero server**: la pagina legge un file JSON e disegna. Niente API a pagamento, niente chiavi.
2. **Coordinate decise prima, non nel browser**: ogni luogo nel JSON ha già `lat`/`lon`. Niente geocodifica a runtime.
   - Per ricavarle: Google Maps → tieni premuto sul punto → copia le coordinate.
3. **Raggruppamento dei marker**: da lontano i punti vicini si fondono in una bolla con il totale (plugin MarkerCluster).
4. **Popup ricchi**: nome, categoria, note e link d'azione.
5. **Filtri a chip** per categoria; si mostrano solo le categorie che hanno almeno un luogo.
6. **Stato nell'URL** (`?cat=spesa`) così un link apre la mappa già filtrata.
7. **Caricamento leggero**: carica il JSON una volta e tienilo in memoria. Lascia lavorare la cache del browser (non usare `cache:"no-store"`).

## 3. Stack

| Cosa | Scelta |
|---|---|
| Libreria mappa | Leaflet 1.9.x (CDN unpkg) |
| Raggruppamento | Leaflet.markercluster 1.5.x |
| Tile (sfondo) | **CARTO Dark Matter** `https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png`: scuro, perfetto per lo stile GTA. Attribuzione: © OpenStreetMap contributors © CARTO |
| GPS | `navigator.geolocation.watchPosition` (su GitHub Pages c'è l'HTTPS, quindi funziona) |
| Navigazione | Deep link a Google Maps e Waze (nessuna API) |
| Dati | `data/luoghi.json` nel repo + luoghi aggiunti dall'utente in `localStorage` (con try/catch) |

**Non** usare direttamente `tile.openstreetmap.org`: la loro policy lo sconsiglia per le app.

## 4. Formato dati – `data/luoghi.json`

```json
[
  {
    "id": "lidl-cuneo",
    "nome": "Lidl",
    "categoria": "spesa",
    "lat": 44.0000,
    "lon": 7.0000,
    "note": "Spesa settimanale – sabato mattina",
    "missione": "spesa-settimana"
  }
]
```

- Le coordinate sopra sono **segnaposto**: vanno sostituite con quelle vere.
- `missione` (facoltativo) collega il luogo a una quest dell'app: dal popup si può aprire la missione.
- Categorie suggerite: `casa`, `lavoro`, `spesa`, `sport`, `svago`, `servizi`.

## 5. Stile GTA

- Sfondo con le tile scure di CARTO; controlli di Leaflet ricolorati con la palette dell'app.
- **Blip**: `L.divIcon` circolari, colore per categoria, emoji o lettera al centro, bordo bianco e glow (`box-shadow` neon).
- **Cluster**: stessa forma del blip, con il numero dei luoghi; il diametro cresce con la radice del conteggio (es. `24 + min(32, √n·5)` px).
- **Posizione utente**: freccia o punto pulsante (animazione CSS) + cerchio di precisione.
- Popup con fondo scuro e titolo in stile "missione".

| Categoria | Colore | Icona |
|---|---|---|
| casa | verde | 🏠 |
| lavoro | blu | 🏭 |
| spesa | giallo | 🛒 |
| sport | rosa | 💪 |
| svago | viola | 🎮 |
| servizi | grigio | ⚙️ |

## 6. Link di navigazione

```js
const linkGoogle = (lat, lon) =>
  `https://www.google.com/maps/dir/?api=1&destination=${lat},${lon}&travelmode=driving`;
const linkWaze = (lat, lon) =>
  `https://waze.com/ul?ll=${lat},${lon}&navigate=yes`;
```

Sul telefono aprono direttamente le app se installate. Aprili con `target="_blank" rel="noopener"`.

## 7. Codice di riferimento (scritto da zero, da adattare)

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.css">
<div id="chips"></div>
<div id="mappa" style="height:70vh"></div>
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://unpkg.com/leaflet.markercluster@1.5.3/dist/leaflet.markercluster.js"></script>
<script>
const CATEGORIE = {
  casa:{label:"Casa", colore:"#3ddc84", icona:"🏠"},
  lavoro:{label:"Lavoro", colore:"#4da3ff", icona:"🏭"},
  spesa:{label:"Spesa", colore:"#ffd23f", icona:"🛒"},
  sport:{label:"Sport", colore:"#ff4fa3", icona:"💪"},
  svago:{label:"Svago", colore:"#a66bff", icona:"🎮"},
  servizi:{label:"Servizi", colore:"#9aa0a6", icona:"⚙️"}
};
const escHtml = s => String(s ?? "").replace(/[&<>"']/g,
  c => ({"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;"}[c]));

const mappa = L.map("mappa").setView([44.384, 7.542], 13); // Cuneo
L.tileLayer("https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png", {
  maxZoom: 19, subdomains: "abcd",
  attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> &copy; <a href="https://carto.com/attributions">CARTO</a>'
}).addTo(mappa);

function blip(cat) {
  const c = CATEGORIE[cat] || CATEGORIE.servizi;
  return L.divIcon({className:"", iconSize:[34,34], iconAnchor:[17,17],
    html:`<div class="blip" style="--c:${c.colore}">${c.icona}</div>`});
}

const gruppo = L.markerClusterGroup({
  showCoverageOnHover:false, maxClusterRadius:40,
  iconCreateFunction: cl => {
    const n = cl.getChildCount(), d = Math.round(24 + Math.min(32, Math.sqrt(n) * 5));
    return L.divIcon({className:"", iconSize:[d,d], iconAnchor:[d/2,d/2],
      html:`<div class="blip cluster" style="width:${d}px;height:${d}px">${n}</div>`});
  }
}).addTo(mappa);

let luoghi = [], filtro = new URLSearchParams(location.search).get("cat") || "";

function leggiLuoghiUtente() {
  try { return JSON.parse(localStorage.getItem("wc_luoghi") || "[]"); } catch { return []; }
}

function popup(l) {
  return `<div class="pop">
    <h3>${CATEGORIE[l.categoria]?.icona || "📍"} ${escHtml(l.nome)}</h3>
    ${l.note ? `<p>${escHtml(l.note)}</p>` : ""}
    <a href="${linkGoogle(l.lat, l.lon)}" target="_blank" rel="noopener">🗺️ Google Maps</a>
    <a href="${linkWaze(l.lat, l.lon)}" target="_blank" rel="noopener">🚗 Waze</a>
  </div>`;
}

function disegna() {
  gruppo.clearLayers();
  luoghi.filter(l => !filtro || l.categoria === filtro)
        .forEach(l => L.marker([l.lat, l.lon], {icon: blip(l.categoria)})
                       .bindPopup(popup(l), {maxWidth: 280}).addTo(gruppo));
}

function chip() {
  const presenti = Object.keys(CATEGORIE).filter(k => luoghi.some(l => l.categoria === k));
  document.getElementById("chips").innerHTML = [["", "Tutti"], ...presenti.map(k => [k, CATEGORIE[k].label])]
    .map(([k, t]) => `<button class="chip${k === filtro ? " on" : ""}" data-k="${k}">${escHtml(t)}</button>`).join("");
}

document.getElementById("chips").addEventListener("click", e => {
  const b = e.target.closest(".chip"); if (!b) return;
  filtro = b.dataset.k;
  const u = new URLSearchParams(location.search);
  filtro ? u.set("cat", filtro) : u.delete("cat");
  history.replaceState(null, "", "?" + u);
  chip(); disegna();
});

// posizione GPS
let io, cerchio;
navigator.geolocation?.watchPosition(p => {
  const ll = [p.coords.latitude, p.coords.longitude];
  if (!io) {
    io = L.marker(ll, {icon: L.divIcon({className:"", html:'<div class="io"></div>', iconSize:[18,18], iconAnchor:[9,9]})}).addTo(mappa);
    cerchio = L.circle(ll, {radius: p.coords.accuracy, color:"#ff2e88", weight:1, fillOpacity:.08}).addTo(mappa);
  } else { io.setLatLng(ll); cerchio.setLatLng(ll).setRadius(p.coords.accuracy); }
}, () => {}, {enableHighAccuracy: true});

(async () => {
  let base = [];
  try { base = await (await fetch("data/luoghi.json")).json(); } catch {}
  luoghi = [...base, ...leggiLuoghiUtente()].filter(l => l.lat != null && l.lon != null);
  chip(); disegna();
})();
</script>
<style>
.blip{width:34px;height:34px;border-radius:50%;display:flex;align-items:center;justify-content:center;
  background:var(--c,#ff2e88);border:2px solid #fff;font-size:16px;box-shadow:0 0 10px var(--c,#ff2e88)}
.blip.cluster{background:#ff2e88;color:#fff;font-weight:800;font-size:13px}
.io{width:18px;height:18px;border-radius:50%;background:#ff2e88;border:3px solid #fff;animation:pulse 1.6s infinite}
@keyframes pulse{0%{box-shadow:0 0 0 0 rgba(255,46,136,.7)}100%{box-shadow:0 0 0 16px rgba(255,46,136,0)}}
.chip.on{background:#ff2e88;color:#fff}
</style>
```

## 8. Funzioni extra (in ordine di priorità)

1. **Aggiungi luogo**: pressione lunga sulla mappa (`mappa.on("contextmenu", …)`) → modulo con nome e categoria → salvato in `localStorage` e incluso nel backup dell'app.
2. **Pulsante "Centra su di me"**.
3. **Collegamento alle missioni**: nel popup, "Apri missione" se `missione` è valorizzato; la missione "Spesa Lidl" mostra il blip del Lidl.
4. **Distanza** dal luogo, calcolata da GPS con `mappa.distance(a, b)`.

## 9. Errori da evitare (lezioni dal sito di riferimento)

- Non usare `cache:"no-store"` sul JSON: rallenta ogni apertura.
- Passa **tutti** i testi dentro l'HTML (anche le etichette dei chip) per la funzione di escape.
- Non mettere luoghi senza coordinate su un punto "di ripiego": se mancano `lat`/`lon`, il luogo non va mostrato e va segnalato.
- Dichiara le variabili di stato **prima** delle callback che le usano.
- Metti sempre l'attribuzione delle tile.

## 10. Criteri di accettazione

- [ ] La mappa si apre su Cuneo, scura, con i blip colorati per categoria.
- [ ] Da zoom basso i luoghi vicini si raggruppano con il conteggio.
- [ ] I chip filtrano e l'URL `?cat=` riapre lo stesso filtro.
- [ ] Il popup ha i pulsanti Google Maps e Waze, che sul telefono avviano la navigazione.
- [ ] Il GPS mostra la posizione con il cerchio di precisione (dopo il permesso).
- [ ] I luoghi aggiunti dall'utente restano dopo il ricaricamento e finiscono nel backup.
- [ ] Funziona su smartphone (CMF Phone) e su iPad.
