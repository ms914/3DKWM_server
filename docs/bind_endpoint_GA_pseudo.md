// GA code für Binde Endpunkt

FUNKTION binde_atome(Atom_A, Atom_B, wolken_A_auswahl, wolken_B_auswahl):

    // --- SETUP & SORTIERUNG ---
    anzahl_wolken = LÄNGE(wolken_A_auswahl)
    lokale_A = HOLE_VEKTOREN(Atom_A, wolken_A_auswahl)
    lokale_B = HOLE_VEKTOREN(Atom_B, wolken_B_auswahl)
    lokale_B = SORTIERE_GEGEN_UEBERKREUZ(lokale_A, lokale_B)

    // Bindungsabstand festlegen (KWM-Constraints)
    WENN anzahl_wolken == 3: abstand = 1.02
    WENN anzahl_wolken == 2: abstand = 1.12
    SONST:                   abstand = 1.25

    ist_A_zentralatom = PRÜFE_OB_BEREITS_GEBUNDEN(Atom_A)

    // --- 1. ACHSE BESTIMMEN ---
    WENN ist_A_zentralatom:
        // In GA: Rotor auf lokale Vektoren anwenden: v_global = Rotor * v_lokal * ~Rotor
        globale_A_auswahl = ROTIERE_MIT_ROTOR(Atom_A.rotor, lokale_A)
        achse = NORMALISIERE( BERECHNE_MITTELPUNKT(globale_A_auswahl) )
    SONST:
        achse = NORMALISIERE(Atom_B.position - Atom_A.position)

    // Translation von B (bleibt klassische Vektoraddition)
    Atom_B.position = Atom_A.position + (achse * abstand)


    // --- 2. ATOM A ROTIEREN (Erstkontakt via Richtungsrotor) ---
    WENN NICHT ist_A_zentralatom:
        zentrum_A_lokal = NORMALISIERE( BERECHNE_MITTELPUNKT(lokale_A) )
        
        // GA-Exzellenz: Rotor direkt aus dem geometrischen Produkt zweier Einheitsvektoren
        // R = sqrt(v2 * v1) -> Erzeugt den idealen Rotor von zentrum_A_lokal nach achse
        Atom_A.rotor = BERECHNE_RICHTUNGS_ROTOR(zentrum_A_lokal, achse)


    // Globale Zielrichtung für B (Invertiertes System von A)
    globale_A_auswahl = ROTIERE_MIT_ROTOR(Atom_A.rotor, lokale_A)
    ziel_B = INVERTIERE(globale_A_auswahl)
    start_B = lokale_B


    // --- 3. SPEZIFISCHE BINDUNGSLOGIK & TORSION (Rotor-Kaskade) ---
    
    // Fall 1: Einfachbindung (Kollinearer Abgleich)
    WENN anzahl_wolken == 1:
        // Direktes geometrisches Produkt der beiden Richtungsvektoren
        Atom_B.rotor = BERECHNE_RICHTUNGS_ROTOR(start_B[0], ziel_B[0])

    // Fall 2: Doppelbindung (Ebenen-Abgleich via Bivektoren)
    WENN anzahl_wolken == 2:
        // 1. Richtungsrotor für die Hauptachse
        R_achse = BERECHNE_RICHTUNGS_ROTOR(start_B[0], ziel_B[0])
        start_B_dreht = ROTIERE_MIT_ROTOR(R_achse, start_B)
        
        // 2. Torsionsrotor über das Äußere Produkt (Bivektor / Ebene)
        ebene_start = NORMALISIERE_BIVEKTOR( start_B_dreht[0] ∧ start_B_dreht[1] )
        ebene_ziel  = NORMALISIERE_BIVEKTOR( ziel_B[0] ∧ ziel_B[1] )
        
        // Rotor, der die beiden Ebenen im Raum zur Deckung bringt
        R_torsion = BERECHNE_EBENEN_ROTOR(ebene_start, ebene_ziel)
        
        // Geometrisches Produkt kombiniert die Rotationen: Erst R_achse, dann R_torsion
        Atom_B.rotor = R_torsion * R_achse

    // Fall 3: Dreifachbindung (Flächen-Teilung & Anti-Umklappen)
    WENN anzahl_wolken == 3:
        zentrum_B_lokal = NORMALISIERE( BERECHNE_MITTELPUNKT(start_B) )
        zentrum_B_ziel  = NORMALISIERE( BERECHNE_MITTELPUNKT(ziel_B) )
        
        // Basis-Rotor (bringt die "Gesichter" zueinander)
        basis_rotor = BERECHNE_RICHTUNGS_ROTOR(zentrum_B_lokal, zentrum_B_ziel)
        
        // --- ANTI-UMKLAPP-SCHUTZ (In GA rein geometrisch über das Skalarprodukt) ---
        zentrum_B_test = ROTIERE_MIT_ROTOR(basis_rotor, zentrum_B_lokal)
        WENN (zentrum_B_test · achse) > 0:
            // Schnitzel-Flip: 180° Drehung um eine orthogonale Achse.
            // In GA ist das ein reiner Bivektor-Rotor (Exponentialform einer Ebene)
            ortho_achse = NORMALISIERE( KREUZPRODUKT(achse, EINHEITSVEKTOR_X) )
            I_flip = EXPLIZITER_180_GRAD_ROTOR(ortho_achse)
            basis_rotor = I_flip * basis_rotor

        // --- TORSIONS-KORREKTUR (-60° auf Lücke) ---
        // Erzeuge das Bivektor-Drehelement direkt auf der Bindungsachse
        // R = cos(theta/2) - sin(theta/2) * B, wobei B der duale Bivektor zur Achse ist
        R_torsion_60 = ERZEUGE_AXIALEN_ROTOR(achse, 60_GRAD)
        
        Atom_B.rotor = R_torsion_60 * basis_rotor

    RÜCKGABE Atom_A, Atom_B
    ´´´
