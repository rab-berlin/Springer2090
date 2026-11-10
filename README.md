# Springer2090

Das Springerproblem (Knight's Tour) für den Busch Microtronic 2090 mit 8x8-LED-Matrix und 4x4-Keypad: Der Springer soll jedes Feld der Matrix genau einmal betreten.

- 246 Programmschritte, 10 frei
- Datei: [springer.mic](springer.mic) (Version 09)

## Spiel

- **Start:** Das Startfeld ist zufällig.
- **Anzeige:** Besuchte Felder leuchten, das aktuelle Feld blinkt.
- **Züge:** Ungültige Züge (über den Rand oder auf ein besuchtes Feld) werden ignoriert.
- **Spielende:** Nach jedem Zug prüft das Programm, ob es noch einen Zug gibt. Wenn nicht, ist das Spiel zu Ende, und die Anzeige zeigt die besuchten Felder. 64 heißt gewonnen.

## Bedienung

Die Tasten sind wie die Sprungrichtungen auf dem Keypad angeordnet (0-3 untere Reihe, C-F obere Reihe):

| Taste | Sprung |
|---|---|
| D / E | 1 links bzw. rechts, 2 hoch |
| 8 / B | 2 links bzw. rechts, 1 hoch |
| 4 / 7 | 2 links bzw. rechts, 1 runter |
| 1 / 2 | 1 links bzw. rechts, 2 runter |
| 0 (halten) | Punkte anzeigen, loslassen spielt weiter |

Die Taste halten, bis der Springer springt.

## Start

Matrix einschalten (siehe unten), `HALT` `NEXT` `00` `RUN`. Das Programm löscht alle Register selbst, ein RESET ist nicht nötig.

## Technik: das Brett ist das Bild

Das Brett liegt in den Speicherregistern M0-MF genau im Format der Matrix: Zeile y ist M(2y) links (Spalte 1-4) und M(2y+1) rechts (Spalte 5-8), ein Bit je Feld. Ein besuchtes Feld ist ein gesetztes Bit. Das Programm schickt dieses Brett nach jeder Änderung als Bild an die Matrix, ein zweiter Bildspeicher entfällt.

| Register | Bedeutung |
|---|---|
| R8 / R9 | X (Spalte 0-7, 0 = links) / Y (Zeile 0-7, 0 = unten) |
| RA | BLINK: 1 = aktuelles Feld gerade dunkel |
| RB | GEHALTEN (1 = Taste war schon gedrückt); 8-F = Prüfung der Zugmöglichkeiten |
| RC / RD | Zähler Einer / Zehner (besuchte Felder, dezimal) |
| RE | P: Registernummer des Feldes (0-F), sonst Hilfsregister |
| RF | Summand für den Zugriff, danach der neue Wert |
| R0 | Hilfsregister (Tasten, Kopie von P) |
| R1 / R2 | P und Maske K des aktuellen Feldes |
| R3 / R4 | alte Lage X, Y |
| R5 / R6 | dx, dy des Zuges |
| R7 | H: welche Brett-Hälfte gerade in M0-M7 liegt (0 = unten, 8 = oben) |

## Matrixprotokoll

Das Programm arbeitet mit dem [ESP2090 Studio](https://github.com/rab-berlin/ESP2090) im Blockmodus, die Matrix schläft zwischen den Bildern:

1. **Wecken:** Das Programm legt F an und hält es, bis der ESP mit REQ antwortet.
2. **Bild:** Dann sendet es direkt ACK 1, genau einen Befehl, die 16 Register und zum Schluss 0.
3. **Schlafen:** Danach schläft der ESP von selbst wieder ein. Er setzt kein REQ und beachtet die Ausgänge nicht, so sind Eingang 1 und die Ausgänge frei für den Keypad-Scan.

Weil das F stehen bleibt, bis der ESP antwortet, geht kein Wecken verloren, auch wenn er gerade mit der Weboberfläche beschäftigt ist.

Auf der Weboberfläche „LED on“ drücken, im [Emulator2090](https://github.com/rab-berlin/Emulator2090) die Matrix mit `m` einschalten (Modus ESP). Das Programm braucht die Firmware mit „ein Befehl je Wecken“ (esp2090-firmware ab PR #38).

## Wie es gemacht ist

Der 2090 hat keine indirekte Adressierung, rechnet nur mit 0-15 und hat nur ein Rücksprungregister. Springer passt trotzdem in 246 Schritte:

### Position in Registernummer und Maske

`Pos` rechnet X und Y in das Matrixregister P = 2·Y + (X ≥ 4) und die Maske K = 1 << (X mod 4) um. Das Aufdoppeln für die Zeile ist ein `ADD E,E`, das Ergebnis von X ≥ 4 holt `CMPI` über den Carry, und `ADC` addiert ihn dazu. Die Maske entsteht ohne Schleife: X mod 4 + 1 ist 1, 2, 3 oder 4. Für 1 und 2 stimmt der Wert schon (1, 2), für 3 und 4 wird 2 abgezogen und zweimal verdoppelt (1 → 4, 2 → 8).

### Setzen, Löschen und Lesen mit einer Routine

`Zugriff` addiert RF zum Brettregister M[P] und gibt den neuen Wert in RF zurück:

- **RF = +K:** Feld einschalten (besucht oder Blink an),
- **RF = −K (0 − K):** Feld ausschalten (Blink aus),
- **RF = 0:** nur lesen. Das anschließende `AND K` zeigt, ob das Feld schon besucht ist.

Dass die Bits sich nie überlappen, hält das Programm selbst sicher: gesetzt wird nur auf leere, gelöscht nur auf gesetzte Felder. Deshalb gibt es keinen Übertrag.

### Zwei Brett-Hälften, ein Fenster

R0-R7 sind das Fenster auf M0-M7. Die obere Brett-Hälfte (M8-MF) kommt mit der Folge `EXRL EXRM EXRA EXRL EXRM` an dessen Stelle und liegt dann in M0-M7. `Zugriff` merkt sich in R7, welche Hälfte gerade unten liegt, und tauscht nur, wenn die andere gebraucht wird. Bei Zügen in derselben Hälfte entfällt der Tausch ganz. Das Bild wird immer mit der unteren Hälfte in M0-M7 gesendet, dafür sorgt `Anzeige`.

### Zuggültigkeit durch eine Vorzeichen-Beobachtung

Ein Zug addiert dx und dy zu X und Y. Negative Werte erscheinen im 4-Bit-Register als 8-F, und alles ab 8 liegt auch über dem Brett. Ein einziger Vergleich (`CMPI 7`, Carry) fängt deshalb beide Ränder in einem Test ab, links wie rechts, oben wie unten. Wurde das Ziel besucht oder liegt es außerhalb, stellt das Programm die alte Lage wieder her und tut so, als wäre nichts gewesen.

### Das Keypad als Sprungkarte

Die acht Sprünge liegen auf dem Keypad wie auf dem Brett: Die Tasten D, E, 8, B, 4, 7, 1, 2 umkreisen die Mitte. Das Programm fragt die vier Reihen nacheinander ab (`DOT`, `DIN`) und leitet aus Reihe und Spalte dx und dy ab. Die Zeile entscheidet über dy (−2, −1, +1, +2), die Spalte über dx. Taste 0 ist die einzige Taste, die als Bit 1 mit `SHR` in den Carry geschoben wird und zur Punkteanzeige führt.

### Ein Zug ist ein Tastendruck

Das Register GEHALTEN merkt sich, dass die Taste schon gedrückt war. Eine gehaltene Taste wird deshalb wie keine Taste behandelt und löst keinen zweiten Sprung aus. Zwischen den Tastenabfragen blinkt das aktuelle Feld: Bei jedem Durchlauf wird es mit `Zugriff` ein- oder ausgeschaltet und das Bild neu gesendet.

### Gibt es noch einen Zug? Acht Richtungen durch Drehen

Nach jedem Zug prüft das Programm, ob der Springer überhaupt noch ziehen kann. Es probiert die acht Sprünge nicht aus einer Tabelle, sondern erzeugt sie durch Rechnen:

1. Start mit der Richtung des letzten Zuges (dx, dy).
2. Dreimal um 90° drehen: (dx, dy) → (−dy, dx). Das sind vier Richtungen.
3. Dann dx und dy vertauschen und noch dreimal drehen: die anderen vier.

Die Prüfung läuft über denselben Code wie ein Tastendruck (`Zug`). Das Register GEHALTEN zählt dabei von F bis 8 und zeigt an, dass geprüft wird. Die Gegenrichtung des letzten Zuges führt zurück auf das Feld, von dem der Springer kam, und ist immer besucht. Sie wird einfach mitgeprüft. Findet sich irgendwo ein freies Feld, geht das Spiel weiter. Sind alle acht gesperrt, ist es zu Ende.

### Zähler ohne Umrechnung

Der Zähler der besuchten Felder ist von Anfang an dezimal: `ADDI 1,C`, bei Überlauf über 9 wird C auf 0 gesetzt und D erhöht. So braucht die Anzeige kein `HXDZ`, und 64 steht als `64` da.

### Punkte anzeigen

Taste 0 gehalten schaltet die Anzeige mit `DISP 2,C` ein, solange man sie hält. Beim Loslassen geht `DISOUT` wieder an, weil die Matrix ihr Bild nur im Takt von 9,3 ms je Befehl annimmt.

### Startfeld

`RND` legt drei Ziffern in D, E, F ab. Das Programm nimmt D und E, maskiert sie mit 7 und hat damit X und Y des Startfelds. Weil `RND` die Register D-F überschreibt, kommt der Zufall vor allen anderen Einstellungen.
