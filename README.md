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

## Matrixprotokoll

Das Programm arbeitet mit dem [ESP2090 Studio](https://github.com/rab-berlin/ESP2090) im Blockmodus, die Matrix schläft zwischen den Bildern:

1. **Wecken:** Das Programm legt F an und hält es, bis der ESP mit REQ antwortet.
2. **Bild:** Dann sendet es direkt ACK 1, genau einen Befehl, die 16 Register und zum Schluss 0.
3. **Schlafen:** Danach schläft der ESP von selbst wieder ein. Er setzt kein REQ und beachtet die Ausgänge nicht, so sind Eingang 1 und die Ausgänge frei für den Keypad-Scan.

Weil das F stehen bleibt, bis der ESP antwortet, geht kein Wecken verloren, auch wenn er gerade mit der Weboberfläche beschäftigt ist.

Auf der Weboberfläche „LED on“ drücken, im [Emulator2090](https://github.com/rab-berlin/Emulator2090) die Matrix mit `m` einschalten (Modus ESP). Das Programm braucht die Firmware mit „ein Befehl je Wecken“ (esp2090-firmware ab PR #38).
