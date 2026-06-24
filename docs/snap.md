
## Der neue UI/UX-Workflow

1. **Der "Snap-Modus" (Vorauswahl):** Wenn der Nutzer Atom B nah an Atom A heranschiebt, sucht das Frontend nach den *n* am nächsten beieinander liegenden Wolken-Paaren.
2. **Visuelles Alignment (Vorschau):** Das Frontend zeigt dem Nutzer temporär an, was passieren *würde* (z. B. durch gestrichelte Linien zwischen den Wolken oder ein gelbes Leuchten). Es sperrt sich aber noch nicht fest.
3. **Die finale Aktion:** Erst wenn der Nutzer mit der Vorschau zufrieden ist und auf **"Verbinden"** klickt, friert der Server die Bindung (Einfach, Doppel oder Dreifach) permanent ein.

---

## Modifizierter Pseudocode

### 1. Frontend: Kontinuierliche Erkennung (Sammelt Kandidaten)

Das Frontend ermittelt jetzt dynamisch, wie viele Wolken überlappen, und speichert diese als "aktuelle Auswahl" im UI-Zustand.

```text
// Globaler Zustand im Frontend
KANDIDATEN_FUER_BINDUNG = { atom_A_id: null, atom_B_id: null, wolken_A: [], wolken_B: [] }

FUNKTION update_interaktion():
    WENN Nutzer_zieht_Atom(atom_B):
        aktualisiere_position_und_rotation_via_maus(atom_B)
        
        // Finde heraus, welche Wolken sich gerade nahe sind
        KANDIDATEN_FUER_BINDUNG = finde_naechste_wolken_paare(atom_B)
        
        // Visuelles Feedback: Zeige Linien zwischen den potenziellen Bindungspartnern
        ZEIGE_BINDUNGS_VORSCHAU(KANDIDATEN_FUER_BINDUNG)
        
        // Aktiviere den Button im UI, wenn mindestens 1 Paar nah genug ist
        WENN LÄNGE(KANDIDATEN_FUER_BINDUNG.wolken_A) > 0:
            AKTIVIERE_BUTTON("Verbinden")
        SONST:
            DEAKTIVIERE_BUTTON("Verbinden")

```

Die Erkennungslogik filtert die Wolken nach Distanz. Da du bis zu 3 Bindungen erlaubst, greift sich das System einfach alle Paare ab, die unter dem Schwellenwert liegen:

```text
FUNKTION finde_naechste_wolken_paare(atom_B):
    MAX_SNAP_DISTANZ = 0.4
    temporaere_liste = []

    FUER JEDES atom_A IN WELT_ZUSTAND_CLIENT:
        WENN atom_A == atom_B: WEITER
        
        FUER JEDE w_A IN atom_A.wolken:
            FUER JEDE w_B IN atom_B.wolken:
                WENN w_A.frei UND w_B.frei:
                    dist = ABSTAND(w_A.globale_pos, w_B.globale_pos)
                    WENN dist < MAX_SNAP_DISTANZ:
                        temporaere_liste.HINZUFUEGEN({a_id: w_A.id, b_id: w_B.id, distanz: dist})
    
    // Sortiere nach kleinster Distanz und nimm die besten (max. 3)
    SORTIERE_NACH_DISTANZ_AUFSTEIGEND(temporaere_liste)
    beste_paare = NEHME_ERSTE_N_ELEMENTE(temporaere_liste, max_anzahl=3)
    
    RÜCKGABE Strukturierte_IDs(beste_paare)

```

### 2. Frontend: Button-Klick Event

Wenn der Button gedrückt wird, wird der exakte Zustand der Vorschau eingefroren und an den Server geschickt. Damit entfällt jegliches Rätselraten.

```text
EVENT_HANDLER on_verbinden_button_click():
    WENN KANDIDATEN_FUER_BINDUNG valide ist:
        SENDE_AN_SERVER_ENDPOINT("/binde_atome", {
            atom_A_id: KANDIDATEN_FUER_BINDUNG.atom_A_id,
            atom_B_id: KANDIDATEN_FUER_BINDUNG.atom_B_id,
            wolken_A_ids: KANDIDATEN_FUER_BINDUNG.wolken_A,
            wolken_B_ids: KANDIDATEN_FUER_BINDUNG.wolken_B
        })

```

---

## Warum das die User Experience (UX) massiv verbessert:

* **Volle Kontrolle bei Mehrfachbindungen:** Ein Kohlenstoffatom kann sich einem anderen nun "schräg" nähern. Der Nutzer sieht im Frontend z. B.: *„Ah, im Moment erkennt das System nur eine Doppelbindung (2 Linien leuchten). Ich drehe das Atom noch ein Stück... Jetzt leuchten 3 Linien! Perfekt.“* -> **Klick auf Verbinden.**
* **Kein "Zappeln" (Jittering):** Weil das mathematische Snapping erst nach dem Klick ausgeführt wird, springt das Atom während des Ziehens nicht unkontrolliert zwischen den Geometrien hin und her. Das Bewegen der Atome im Three.js-Viewport bleibt vollkommen flüssig.
