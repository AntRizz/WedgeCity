# Wedge City — specifiche

La vita di Antonio a Cuneo come un open world: missioni, mappa, spesa, routine.
"Wedge City" = Cuneo in inglese, con il ritmo di "Vice City". Stile ispirato ai giochi
a mondo aperto, ma grafica originale: niente loghi, font, immagini o nomi di Rockstar.

## Tecnologia
- Una sola pagina `index.html` (HTML, CSS, JavaScript senza build), ospitata su GitHub Pages.
- PWA: `manifest.json` + `sw.js`, si installa sulla schermata Home e funziona offline
  (tranne mappa, percorsi e ricerca indirizzi, che usano la rete).
- Librerie (unpkg): Leaflet 1.9.4, Leaflet.markercluster 1.5.3, maplibre-gl 4.7.1, @maplibre/maplibre-gl-leaflet 0.1.4.
- Servizi esterni gratuiti:
  - mappa: OpenFreeMap (stile dark) tramite MapLibre GL dentro Leaflet; nessuna chiave.

## Dati
- Salvati solo sul telefono (localStorage), chiave `m15`.
- Backup: "Copia backup" e "Importa" (JSON), in fondo alla pagina.
- Struttura: `checks`, `real`, `home`, `spent`, `weights`, `budget`, `custom`, `luoghi`, `coord`. Luoghi fissi in `data/luoghi.json`.
- Nessun dato personale nel repository (il repository è pubblico).

## Navigazione
Barra in basso con 5 sezioni, sotto-schede in alto quando servono:
- **Oggi**: mappa dei 19 giorni (27/11 → 15/12), missioni del giorno divise in mattina, giornata, sera, più le missioni tue.
- **Mappa**: GPS, segnalino, percorso, punti, Google Maps e Waze.
- **Missioni**: Casa, Hobby, Lavoro.
- **Cibo**: Spesa Lidl, Menù, Meal prep.
- **Tu**: Peso, Routine, Premi.

## Gioco
- XP per ogni missione; livello ogni 250 XP; gradi: Pivello, Galoppino, Uomo di fiducia,
  Luogotenente, Boss del quartiere, Leggenda di Cuneo (3 livelli ciascuno).
- Gettoni: 1 ogni 10 XP, si spendono nel negozio premi (con annulla).
- Serie: giorni consecutivi con almeno il 60% fatto; il giorno in corso non la rompe.
- Contatore € in alto: risparmio rispetto alla spesa solita (50 €/settimana).
- Avviso "Missione completata" a ogni spunta.

## Mappa
Vedi **SPEC-MAPPA.md** (specifica di riferimento e scostamenti). In breve: Leaflet + OpenFreeMap dark,
blip per categoria con raggruppamento, chip filtro con `?cat=`, popup con Google Maps e Waze,
GPS con punto pulsante e cerchio di precisione, pressione lunga per aggiungere un luogo.

## Routine (entrata al lavoro 7:30, uscita 17:30)
6:15 sveglia · 6:25 colazione · 7:00 partenza · pausa camminata · 18:00 rientro (allenamento lun/mer/ven) ·
19:15 casa · 19:40 cena · 21:10 inglese · 21:40 prepara domani · 22:15 schermi spenti · 22:30 a letto.

## Feedback
- Pulsante "Feedback" sempre visibile: apre una issue GitHub precompilata con etichetta `feedback`,
  tipo (Idea, Qualcosa non va, Contenuto), scheda attiva, data e dimensioni schermo.
- Le issue sono il database dei feedback: Claude le legge e le chiude quando le ha risolte.

## Fuori dallo scopo (per ora)
- Notifiche e promemoria: richiedono un server di push; si usano sveglie e promemoria del telefono.
- Sincronizzazione tra dispositivi: si usa il backup.
