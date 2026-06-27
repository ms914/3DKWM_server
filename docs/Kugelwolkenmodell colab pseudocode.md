Hier ist der strukturierte **Pseudocode**, der die Kernlogik deines molekularen Baukastens (insbesondere die geometrische Ausrichtung und das Verknüpfen von Bindungen im Backend) abstrahiert und algorithmisch zusammenfasst.  
Dieser Pseudocode konzentriert sich auf das Zusammenspiel zwischen der mathematischen Rotationsmatrix und der Verwaltung der chemischen Lewis-Kugelwolken.

## **1\. Datenstrukturen**

Plaintext  
Struktur Wolke:  
    lokal: Vektor3D         // Relative Richtung vom Atomkern (normiert)  
    elektronen: Ganzzahl    // 1 \= Radikal, 2 \= freies Paar, 3 \= Bindung  
    partner: String         // ID des gebundenen Partner-Atoms oder NULL

Struktur Atom:  
    symbol: String          // "C", "H" oder "O"  
    position: Vektor3D      // Absolute Koordinaten im Raum  
    wolken: Liste von Wolke

## **2\. Mathematische Hilfsfunktion: Ausrichtung**

Plaintext  
Funktion berechneAusrichtungsMatrix(startVektor, zielVektor):  
    v\_from \= normiere(startVektor)  
    v\_to \= normiere(zielVektor)  
      
    Wenn v\_from nahe zu v\_to:  
        Gibt Einheitsmatrix(3x3) zurück  
          
    Wenn v\_from nahe zu \-v\_to (exakt entgegengesetzt):  
        ortho \= FindeOrthogonalenVektor(v\_from)  
        achse \= normiere(Kreuzprodukt(v\_from, ortho))  
        Gibt RotationsmatrixUmAchse(achse, 180\_Grad) zurück  
          
    // Standard-Rotation über Kreuz- und Skalarprodukt (Rodrigues-Formel)  
    v \= Kreuzprodukt(v\_from, v\_to)  
    c \= Skalarprodukt(v\_from, v\_to)  
    s \= Länge(v)  
      
    K \= SchiefsymmetrischeMatrix(v)  
    R \= Einheitsmatrix \+ K \+ (K \* K) \* ((1 \- c) / (s^2))  
      
    Gibt R zurück

## **3\. Hauptalgorithmus: Bindung knüpfen (/binde)**

Plaintext  
Funktion bindeAtome(atom\_A\_id, atom\_B\_id):  
    // 1\. Atome aus dem globalen Zustand laden  
    atom\_A \= WELT\_ZUSTAND\[atom\_A\_id\]  
    atom\_B \= WELT\_ZUSTAND\[atom\_B\_id\]  
      
    // 2\. Aktuelle Bindungsordnung ermitteln  
    bereits\_gebunden \= ZähleWolkenMitPartner(atom\_A, atom\_B\_id)  
    ziel\_ordnung \= bereits\_gebunden \+ 1  
      
    Wenn ziel\_ordnung \> 3:  
        Gibt Fehler("Maximale Bindungsordnung überschritten") zurück

    // 3\. Freie Wolken für die neue Bindung finden (Priorität: Radikal vor Paar)  
    wolke\_A \= FindeFreieWolke(atom\_A, bevorzugte\_elektronen=1)  
    wolke\_B \= FindeFreieWolke(atom\_B, bevorzugte\_elektronen=1)  
      
    Wenn wolke\_A ist NULL oder wolke\_B ist NULL:  
        Gibt Fehler("Keine freien Elektronenwolken verfügbar") zurück  
          
    // 4\. Bindungsabstand anhand der Elemente und Ordnung bestimmen  
    abstand \= BestimmeAbstand(atom\_A.symbol, atom\_B.symbol, ziel\_ordnung)  
      
    // 5\. Geometrische Achse berechnen (Mittelwert bisheriger Bindungen \+ neue Wolke)  
    beteiligte\_wolken\_A \= FindeAlleWolkenMitPartner(atom\_A, atom\_B\_id) \+ wolke\_A  
    lokaler\_vektor\_A \= SummiereLokaleVektoren(beteiligte\_wolken\_A)  
    achse\_A \= normiere(lokaler\_vektor\_A)  
      
    // 6\. Atom B im Raum verschieben  
    atom\_B.position \= atom\_A.position \+ (achse\_A \* abstand)  
      
    // 7\. Rotations-Ausrichtung für Atom B berechnen  
    ziel\_achse\_B \= \-achse\_A  
    beteiligte\_wolken\_B \= FindeAlleWolkenMitPartner(atom\_B, atom\_A\_id) \+ wolke\_B  
    start\_vektor\_B \= SummiereLokaleVektoren(beteiligte\_wolken\_B)  
    start\_achse\_B \= normiere(start\_vektor\_B)  
      
    // Basis-Rotationsmatrix ermitteln  
    R\_matrix \= berechneAusrichtungsMatrix(start\_achse\_B, ziel\_achse\_B)  
      
    // 8\. Stereochemische Torsion anwenden (Konformations-Erzwingung)  
    Wenn ziel\_ordnung \== 2:  
        // Erzwinge planare/ekliptische Ausrichtung für Doppelbindungen  
        R\_torsion \= RotationsmatrixUmAchse(achse\_A, 35.26\_Grad)  
        R\_matrix \= MultipliziereMatrizen(R\_torsion, R\_matrix)  
    Sonst Wenn ziel\_ordnung \== 3:  
        // Dreifachbindung versetzt ausrichten  
        R\_torsion \= RotationsmatrixUmAchse(achse\_A, 60\_Grad)  
        R\_matrix \= MultipliziereMatrizen(R\_torsion, R\_matrix)  
          
    // 9\. Lokale Wolken-Vektoren von Atom B transformieren  
    Für jede w in atom\_B.wolken:  
        w.lokal \= MultipliziereMatrixMitVektor(R\_matrix, w.lokal)  
          
    // 10\. Elektronen- und Partnerstatus aktualisieren  
    wolke\_A.elektronen \= 3  // Status: "Geteiltes Paar / Bindung"  
    wolke\_A.partner \= atom\_B\_id  
    wolke\_B.elektronen \= 3  
    wolke\_B.partner \= atom\_A\_id  
      
    Gibt Erfolg(ziel\_ordnung) zurück  
