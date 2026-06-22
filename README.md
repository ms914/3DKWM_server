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

- # Projekt: Kugelwolkenmodell 3D (Status: H2O-Prototyp läuft)
- Änderungen Server (kwm_server_H2O.ipynb): Wasserstoff hat nur eine Elektronwolke, die mit dem Kern überlappt
- Änderungen Client three.js (H2O_index.html): Wasserstoff weiss und klein zeichnen, onMouseClick selektiert erstes Atom als Atom A und das zweite als Atom B, anklickbare Wolken müssen beim Zeichnen wieder auf 0 gesetzt werden, Daten müssen zurückgesetzt werden. - -aktualisiere3Dscene(daten): bei jedem Aufruf werden die Wolken und Bindungswolken upgedated mit anklickbareWolken.push(wolkeMesh)
- Reihenfolge der clicks: es muss ein Wasserstoff-Atom an die Geometrie des Sauerstoff gebunden werden, da sonst der Prokrustes Algorithmus nicht funktioniert. Die Reihenfolge wird intern so dargestellt, wenn andersrum geklickt wurde.

- # Projekt: Kugelwolkenmodell (Status: PSE - Prototyp läuft)
- Änderungen Server: Füge Chemie Modul hinzu, Bindungsmodul berechnet Orientierung nach Bindungstyp (1-fach, 2-fach, 3-fach)
- Änderungen Client: Mehrere Atome können der Szene hinzugefügt werden
- Maussteuerung: wähle Bindungswolken aus und verbinde
- Geometrie bei Mehrfachbindungen in Vektoralgebra und GA
- C-C 3-fach Bindung: Orientierung kann flippen, so dass die Bindungswolke aussen liegt (gefixed, SVD Mod)

- # Projekt: Kugelwolkenmodell (Status: PSE - Annäherungsmodul)
- Änderungen Server: 
- Änderungen Client: Orientierung der Atome vor der Bindung frei wählbar (Rotation und Translation), Annäherungsmodul für Bindungen (wählbar per Mausklick oder Verschiebung)
- Übertragung der Logik auf weitere Implementierungen (KWM javascript Module, html-app, python-Frontend)
- Elektronen in Wolken, Annäherungs-Animation, Bananen als Bindungswolken, Schalenmodell, Lewis, Verknüpfung mit Avogadro, hell-dunkel, vorgefertigte Moleküle, Import-, Export als mol File, smiles Input, GA-Logik
