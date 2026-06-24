## 2. Der System-Pseudocode

Dieser Pseudocode dokumentiert die logische Architektur des `/binde`-Endpoints.

1. Setup&Sortierung: Hole lokale Wolkenkoordinaten der angewählten Bindungswolken, sortiere die von B um; prüfe ob berits Bindungen vorliegen denn dann ist A das Zentralatom
2. Achse bestimmen und B auf dieser Achse rotieren und verschieben: Wenn 1 Bindungswolke, dann ist das die Achse, bei mehreren ist die Achse die Mitte zwischen den Wolken
3. Atom A rotieren: Atom A wird zur Achse ausgerichtet, indem der Mittelpunkt der Bindungswolken auf die Achse gedreht wird; B Ritation bestimmen
4. Spezifische Bindungslogik (SVD): je nachdem ob Einfach-, Doppel-, oder Dreifachbindung


```text
FUNKTION binde_atome(Atom_A, Atom_B, wolken_A_auswahl, wolken_B_auswahl):

    // --- SETUP & SORTIERUNG ---
    anzahl_wolken = LÄNGE(wolken_A_auswahl)
    lokale_A = HOLE_VEKTOREN(Atom_A, wolken_A_auswahl)
    lokale_B = HOLE_VEKTOREN(Atom_B, wolken_B_auswahl)
    lokale_B = SORTIERE_GEGEN_UEBERKREUZ(lokale_A, lokale_B)

    WENN anzahl_wolken == 3: abstand = 1.02
    WENN anzahl_wolken == 2: abstand = 1.12
    SONST:                   abstand = 1.25

    ist_A_zentralatom = PRÜFE_OB_BEREITS_GEBUNDEN(Atom_A)

    // --- 1. ACHSE BESTIMMEN ---
    WENN ist_A_zentralatom:
        globale_A_auswahl = ROTIERE_VEKTOREN(Atom_A.rotation, lokale_A)
        achse = NORMALISIERE( BERECHNE_MITTELPUNKT(globale_A_auswahl) )
    SONST:
        achse = NORMALISIERE(Atom_B.position - Atom_A.position)

    // B wird verschoben
    Atom_B.position = Atom_A.position + (achse * abstand)

    // --- 2. ATOM A ROTIEREN (Nur Erstkontakt) ---
    WENN NICHT ist_A_zentralatom:
        zentrum_A_lokal = NORMALISIERE( BERECHNE_MITTELPUNKT(lokale_A) )
        drehachse = KREUZPRODUKT(zentrum_A_lokal, achse)
        winkel = SKALARPRODUKT(zentrum_A_lokal, achse)
        Atom_A.rotation = RODRIGUES_ROTATION(drehachse, winkel)

    // Globale Zielkoordinaten für B fixieren (immer invertiert zu A)
    globale_A_auswahl = ROTIERE_VEKTOREN(Atom_A.rotation, lokale_A)
    ziel_B = INVERTIERE(globale_A_auswahl)
    start_B = lokale_B

    // --- 3. SPEZIFISCHE BINDUNGSLOGIK (SVD) ---
    WENN anzahl_wolken == 1:
        kovarianz = OUTER_PRODUCT(start_B, ziel_B)
        Atom_B.rotation = SVD_ABGLEICH(kovarianz)

    WENN anzahl_wolken == 2:
        normalenvektor_A = KREUZPRODUKT(ziel_B[0], ziel_B[1])
        normalenvektor_B = KREUZPRODUKT(start_B[0], start_B[1])
        FÜGE_HINZU(ziel_B, normalenvektor_A)
        FÜGE_HINZU(start_B, normalenvektor_B)
        
        kovarianz = MATRIX_MULTIPLIKATION(TRANSIONIERE(start_B), ziel_B)
        Atom_B.rotation = SVD_ABGLEICH(kovarianz)

    WENN anzahl_wolken == 3:
        kovarianz = MATRIX_MULTIPLIKATION(TRANSIONIERE(start_B), ziel_B)
        basis_rotation = SVD_ABGLEICH(kovarianz)

        // Anti-Umklapp-Schutz (Schnitzel-Flip)
        zentrum_B_test = BERECHNE_MITTELPUNKT( ROTIERE_VEKTOREN(basis_rotation, start_B) )
        WENN SKALARPRODUKT(zentrum_B_test, achse) > 0:
            flip_matrix = BERECHNE_180_GRAD_ORTHO_ROTATION(achse)
            basis_rotation = MATRIX_MULTIPLIKATION(flip_matrix, basis_rotation)

        // Torsions-Korrektur (60° auf Lücke)
        torsions_matrix = BERECHNE_ACHSEN_ROTATION(achse, 60_GRAD)
        Atom_B.rotation = MATRIX_MULTIPLIKATION(torsions_matrix, basis_rotation)
