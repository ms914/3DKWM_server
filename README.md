# 3DKWM_server

# Projekt: Kugelwolkenmodell 3D (Status: Basis-Prototyp läuft)

## Architektur & Datenfluss
1. **Mathematik (Python):** Definiert Atome als Kern-Koordinaten und 4 Tetraeder-Vektoren (Kugelwolken).
2. **API-Schnittstelle (FastAPI):** - `GET /status` liefert das aktuelle Molekül-JSON.
   - `POST /binde` triggert die Procrustes-SVD-Berechnung im Backend.
   - `POST /reset` setzt den WELT_ZUSTAND auf Startwerte zurück.
3. **Tunnel (Ngrok):** Übersetzt den lokalen Colab-Server in eine öffentliche HTTPS-URL.
4. **Visualisierung (Three.js):** Holt per `fetch` die Daten und zeichnet rote Kerne und blaue/graue Wolken.

## Aktuelle Start-Konfiguration
- Atom A: O_links bei [0, 0, 0]
- Atom B: O_rechts bei [3, 2, 1] (schief im Raum)
- Bindung: Starr verdreht über hartcodierte Wolken-IDs [2, 3] im Backend.
