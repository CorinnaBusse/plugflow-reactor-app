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
- **Photometer:** Zeigt den zuletzt gemessenen Absorptionswert (mit Rauschen).
- **Absorption über Zeit:** Simulierte Messpunkte mit Rauschen; die graue
  Linie markiert die maximale Absorption bei vollständigem Umsatz.
- **Konzentrationsprofil:** Momentaufnahme von Edukt c und Produkt P entlang
  der Rohrlänge.
- **Zoom:** In beiden Diagrammen Bereich aufziehen oder Mausrad verwenden.
- **CSV-Export:** Spalten Zeit, Volumenstrom Pumpe Ester, Volumenstrom Pumpe
  Puffer und gemessene Absorption am Ausgang. Alle übrigen Parameter (Länge,
  Rohrquerschnitt, c_ein_Ester, k, ε, d, Rauschen, Messintervall) stehen als
  Metadaten in den `#`-Kopfzeilen. Deutsches Format (Semikolon, Dezimalkomma).

## Parameter

| Gruppe | Parameter | Bereich | Änderbar |
|---|---|---|---|
| Reaktor | Reaktorlänge L | 10–700 cm | nur vor Start |
| | Konz. Edukt-Stammlösung c_E0 | 0,01–2 mmol/L (Standard 0,2) | nur vor Start |
| | Geschwindigkeitskonstante k | 0,002–0,5 s⁻¹ | nur vor Start |
| Messung | Extinktionskoeffizient ε | 0,1–50 L/(mmol·cm) (Standard 5) | nur vor Start |
| | Schichtdicke Küvette d | 0,1–5 cm (Standard 1) | nur vor Start |
| | Messrauschen σ | 0–3 % (Standard 2) | nur vor Start |
| | Messintervall | 0,5–60 s | nur vor Start |
| Zulauf | Pumpe Ester (Q_E), Pumpe Puffer (Q_Puffer) — Zahleneingabe | 2–150 mL/min | jederzeit |
| Zeitraffer | Beschleunigung | ×1–×100 | jederzeit |

Nach **Start** sind die Reaktor- und Messparameter für den laufenden Versuch
gesperrt (auch während einer Pause). Erst **Neuer Lauf** setzt die Simulation
zurück und gibt sie wieder frei. Die Pumpen bleiben verstellbar, damit sich
z. B. die Totzeit nach einer Änderung des Volumenstroms beobachten lässt.

## Modell

```
∂c/∂t + u·∂c/∂z = −k·c        (Edukt)
∂P/∂t + u·∂P/∂z = +k·c        (Produkt)

u     = (Q_E + Q_Puffer) / A_Rohr
τ     = L / u
c_ein = c_E0 · Q_E / (Q_E + Q_Puffer)
A     = ε · d · P(Ausgang)      (Lambert-Beer)
```

Alle Konzentrationen werden in mmol/L angegeben. Mit den Standardwerten
(c_E0 = 0,2 mmol/L, Q_E = Q_Puffer = 30 mL/min) ergibt sich eine
Eingangskonzentration c_ein = 0,1 mmol/L und bei vollständigem Umsatz eine
Absorption von A = 5 · 1 · 0,1 = 0,5.

- Räumliche Diskretisierung: Finite-Volumen-Verfahren mit 40 Zellen
  (Upwind), explizite Zeitintegration mit automatisch angepasster
  Unterschrittweite.
- Fester Rohrquerschnitt A_Rohr = 0,2 cm².
- Im stationären Zustand gilt c_aus = c_ein · e^(−kτ); die Simulation weicht
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
