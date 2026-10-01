# Rohrreaktor·Monitor — Pfropfenströmungs-Simulation

Interaktive Live-Simulation eines Rohrreaktors (Plug-Flow-Reaktor) mit einer
Reaktion 1. Ordnung E → P, im Corporate Design der TH Nürnberg
(Fakultät Angewandte Chemie). Am Reaktorausgang wird die Absorption des
Produkts „gemessen“ — inklusive einstellbarem Messrauschen und CSV-Export für
die anschließende Auswertung.

**Online:** <https://corinnabusse.github.io/plugflow-reactor-app/>

## Funktionen

- **Reaktorgrafik:** Zwei Pumpen (Edukt E, Puffer) speisen das Rohr; die
  Färbung zeigt die lokale Produktkonzentration, die Küvette den Ausgang.
- **Absorption über Zeit:** Simulierte Messpunkte mit Rauschen; die graue
  Linie markiert die maximale Absorption bei vollständigem Umsatz.
- **Konzentrationsprofil:** Momentaufnahme von Edukt C und Produkt P entlang
  der Rohrlänge.
- **Zoom:** In beiden Diagrammen Bereich aufziehen oder Mausrad verwenden.
- **CSV-Export:** Messdaten (Zeit, gemessene und modellierte Absorption) samt
  Parametern als Kopfzeilen; deutsches Format (Semikolon, Dezimalkomma).

## Parameter

| Gruppe | Parameter | Bereich | Änderbar |
|---|---|---|---|
| Reaktor | Reaktorlänge L | 10–200 cm | nur vor Start |
| | Konz. Edukt-Stammlösung C_E0 | 0,1–2 mol/L | nur vor Start |
| | Geschwindigkeitskonstante k | 0,002–0,5 s⁻¹ | nur vor Start |
| Messung | Extinktionskoeffizient ε | 0,2–5 | nur vor Start |
| | Messrauschen σ | 0–15 % | nur vor Start |
| | Messintervall | 0,5–60 s | nur vor Start |
| Zulauf | Pumpe E (Q_E), Pumpe Puffer (Q_Puffer) | 2–150 mL/min | jederzeit |
| Zeitraffer | Beschleunigung | ×1–×2000 | jederzeit |

Nach **Start** sind die Reaktor- und Messparameter für den laufenden Versuch
gesperrt (auch während einer Pause). Erst **Neuer Lauf** setzt die Simulation
zurück und gibt sie wieder frei. Die Pumpen bleiben verstellbar, damit sich
z. B. die Totzeit nach einer Änderung des Volumenstroms beobachten lässt.

## Modell

```
∂C/∂t + u·∂C/∂z = −k·C        (Edukt)
∂P/∂t + u·∂P/∂z = +k·C        (Produkt)

u     = (Q_E + Q_Puffer) / A_Rohr
τ     = L / u
C_ein = C_E0 · Q_E / (Q_E + Q_Puffer)
A     = ε · P(Ausgang)
```

- Räumliche Diskretisierung: Finite-Volumen-Verfahren mit 40 Zellen
  (Upwind), explizite Zeitintegration mit automatisch angepasster
  Unterschrittweite.
- Fester Rohrquerschnitt A_Rohr = 0,2 cm².
- Im stationären Zustand gilt C_aus = C_ein · e^(−kτ); die Simulation weicht
  davon um weniger als 0,5 % ab.

## Entwicklung

Voraussetzung: Node.js 20 oder neuer.

```bash
npm install
npm run dev       # Entwicklungsserver
npm run build     # Produktions-Build nach dist/
npm run preview   # Build lokal ansehen
```

Technik: React 18, Vite 5, Tailwind CSS 4, Recharts. Der gesamte
Anwendungscode liegt in [`src/App.jsx`](src/App.jsx).

## Deployment

Jeder Push auf `main` baut die App per GitHub Actions
([`.github/workflows/deploy.yml`](.github/workflows/deploy.yml)) und
veröffentlicht sie auf GitHub Pages. Der Basispfad `/plugflow-reactor-app/`
ist in [`vite.config.js`](vite.config.js) gesetzt und muss bei einer
Umbenennung des Repositorys angepasst werden.
