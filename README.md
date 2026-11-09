# Springer2090

Das Springerproblem (Knight's Tour) für den Busch Microtronic 2090 mit 8x8-LED-Matrix und 4x4-Keypad: Der Springer soll jedes Feld der Matrix genau einmal betreten.

- 255 Programmschritte, 1 frei
- Datei: [springer.mic](springer.mic) (Version 05)

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

Matrix einschalten, `HALT` `NEXT` `00` `RUN`. Das Programm löscht alle Register selbst, ein RESET ist nicht nötig.

## Matrixprotokoll

Das Programm verwendet das **frühere Blockprotokoll** der Matrix:
- Bereit-Signal auf Eingang 1
- Startnibble 1, 16 Werte, Endnibble 0
- ein F von 15-25 ms schaltet die Matrix um

Weil das Keypad dieselben Leitungen benutzt, schaltet das Programm die Matrix nur für die Bildausgabe ein.

Im [Emulator2090](https://github.com/rab-berlin/Emulator2090) die Matrix mit `m` auf `alt` stellen. Für das aktuelle Protokoll des [ESP2090 Studios](https://github.com/rab-berlin/ESP2090) muss das Programm noch umgestellt werden.
