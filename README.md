# Springer2090

Das Springerproblem (Knight's Tour) für den Busch Microtronic 2090 mit 8x8-LED-Matrix und 4x4-Keypad: Der Springer soll jedes Feld der Matrix genau einmal betreten.

- 255 Programmschritte, 1 frei
- Datei: [springer.mic](springer.mic) (Version 08)

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

## Matrixprotokoll

Das Programm arbeitet mit dem [ESP2090 Studio](https://github.com/rab-berlin/ESP2090) (Blockmodus mit Schlafmodus):

- **Schlafen:** Zwischen den Bildern schläft die Matrix (ACK 15). Der ESP setzt dann kein REQ und ignoriert die Ausgänge, so sind Eingang 1 und die Ausgänge frei für den Keypad-Scan.
- **Wecken und Bild:** Für ein Bild legt das Programm F an und hält es, bis der ESP mit REQ antwortet. Dann sendet es direkt ACK 1, genau einen Befehl und die 16 Register. So geht kein Wecken verloren, egal wie spät der ESP nachsieht (z. B. bei offener Weboberfläche).
- **Wieder schlafen:** Danach setzt das Programm die Ausgänge auf 0 und legt die Matrix wieder schlafen (ACK 15, nach zwei Schritten eine 0).

Auf der Weboberfläche „LED on“ drücken, im [Emulator2090](https://github.com/rab-berlin/Emulator2090) die Matrix mit `m` einschalten (Modus ESP). Die Matrix muss vor dem Start eingeschaltet sein.

Bis Version 05 lief das Programm mit dem früheren Blockprotokoll (Startnibble 1, Endnibble 0, Umschaltsignal F). Version 06 hat eine gemeinsame Routine für Schlafen und Wecken und ist dadurch 3 Schritte kürzer; Version 07 wartet nach dem Wecken auf REQ (+2).

Das Programm braucht die Firmware des ESP2090 Studios mit dem Weck-Handshake (gehaltenes F).
