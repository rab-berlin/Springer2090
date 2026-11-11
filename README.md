# Springer2090

Das Springerproblem (Knight's Tour) für den Busch Microtronic 2090 mit 8x8-LED-Matrix: Der Springer soll jedes Feld der Matrix genau einmal betreten. Bedient wird mit den Hex-Tasten des Microtronic, ein Keypad ist nicht nötig.

- 211 Programmschritte, 45 frei
- Datei: [springer.mic](springer.mic)

## Spiel

- **Start:** Das Startfeld ist zufällig.
- **Farben:** Das aktuelle Feld leuchtet gelb, die besuchten Felder rot. Nichts blinkt.
- **Züge:** Der Springer springt wie beim Schach. Ungültige Züge (über den Rand oder auf ein besuchtes Feld) werden nicht ausgeführt.
- **Hinweise:** Jede ungültige Taste zeigt alle möglichen Zielfelder grün. Sie bleiben stehen, bis ein gültiger Zug ausgeführt ist.
- **Anzeige:** Das Display zeigt die besuchten Felder (dezimal), solange das Programm auf eine Taste wartet.
- **Spielende:** Nach jedem Zug prüft das Programm, ob es noch einen Zug gibt. Wenn nicht, ist das Spiel zu Ende, und die Anzeige zeigt die besuchten Felder. 64 heißt gewonnen.

## Bedienung

Die acht Sprungtasten sind so angeordnet, wie die Sprünge um die Mitte eines 4x4-Feldes liegen (0-3 untere Reihe, C-F obere Reihe):

| Taste | Sprung |
|---|---|
| D / E | 1 links bzw. rechts, 2 hoch |
| 8 / B | 2 links bzw. rechts, 1 hoch |
| 4 / 7 | 2 links bzw. rechts, 1 runter |
| 1 / 2 | 1 links bzw. rechts, 2 runter |

Jede andere Taste (0 3 5 6 9 A C F) zeigt die Hinweise. Im [Emulator2090](https://github.com/rab-berlin/Emulator2090) sind es die Tasten `0`-`9` und `a`-`f`.

## Start

Matrix einschalten („LED on“ in der Weboberfläche, im Emulator Taste `m`), dann `HALT` `NEXT` `00` `RUN`. Das Programm löscht alle Register und die Matrix selbst, ein RESET ist nicht nötig.

## Technik: Brett und Bild getrennt

**Schreibweise in diesem README:** Ein Register heißt `R0` bis `RF` (Speicher: `M0` bis `MF`), ein Wert beginnt mit `#`, zum Beispiel `#3` oder `#F`. `RF` ist also ein Register, `#F` die Zahl fünfzehn. Die Programmzeilen sind hier mit `R` und `#` geschrieben, in der Datei `springer.mic` stehen die Nummern ohne Zusatz: `MOVI #1,RA` heißt dort `MOVI 1,A`.

Das Programm schickt kein Bild mehr. Der ESP merkt sich das Bild, und das Programm ändert nur einzelne Pixel mit Zeichenbefehlen. Das Brett liegt trotzdem im Programm, in den Speicherregistern `M0`-`MF`, ein Bit je Feld (besucht oder nicht). Der ESP lässt sich nicht fragen, welche Felder besucht sind.

| Register | Bedeutung |
|---|---|
| `R3` / `R4` | X / Y des aktuellen Feldes (`#0`-`#7`, X von links, Y von unten) |
| `R5` / `R6` | dx, dy des Sprunges |
| `R8` / `R9` | X / Y des Zielfeldes |
| `R1` / `R2` | Nummer `P` des Brettregisters (`#0`-`#F`) und Maske `K` des zuletzt berechneten Feldes |
| `R7` | gefunden: im Durchlauf gab es mindestens ein freies Zielfeld |
| `RA` | Hinweise sichtbar (`#1`) |
| `RB` | `#0` = einzelner Zug, `#F` bis `#8` = Durchlauf über die 8 Richtungen |
| `RC` / `RD` | Zähler Einer / Zehner (besuchte Felder dezimal) |
| `RE` / `RF` | ACK / erstes Nibble des Matrix-Befehls (sonst Hilfsregister) |
| `R0` | Hilfsregister |

## Matrixprotokoll

Das Programm arbeitet mit dem [ESP2090 Studio](https://github.com/rab-berlin/ESP2090) im Zeichenmodus. Die Matrix bleibt dauernd wach, Schlafen und Wecken braucht das Programm nicht, weil die Eingänge für die Hex-Tasten nicht belegt sind. Es benutzt vier Befehle:

| Befehl | Wofür |
|---|---|
| ACK 3 (Zeichenart 3) | am Start alle Pixel aus |
| ACK 14 | Farbe wählen: `#1` rot, `#3` gelb, `#4` grün |
| ACK 4 | Punkt: mit Pixel an (X + `#8`) oder nur die Farbe (Zeichenart `#0`) |
| ACK 13 | Stapel der Bitmatrix: push (`#2`) und pop (`#6`) |

Auf der Weboberfläche „LED on“ drücken, im Emulator die Matrix mit `m` einschalten. Die Matrix muss vor dem Start an sein: Das Programm wartet gleich am Anfang auf die erste Anfrage (REQ) der Matrix.

## Die Warnsdorff-Regel

Ein **Rösselsprung** (Springertour) besucht alle 64 Felder genau einmal. Probieren ohne Plan scheitert fast immer irgendwo um das 50. Feld: Der Springer steht auf einem Feld, von dem aus kein unbesuchtes Feld mehr erreichbar ist. Eine einfache Faustregel vermeidet das meistens:

> **Springe immer auf das Zielfeld, von dem aus es danach die wenigsten weiteren freien Züge gibt.**

Das ist die Regel von Warnsdorff.

### So wendest du sie an

1. Drücke eine ungültige Taste. Die möglichen Zielfelder leuchten grün.
2. Zähle für jedes grüne Feld, wie viele **unbesuchte** Felder ein Springer von dort noch erreichen würde. Das Feld, auf dem du gerade stehst, und alle roten Felder zählen nicht mit.
3. Wähle das grüne Feld mit der **kleinsten** Zahl. Bei Gleichstand nimmst du irgendeines. Dann ziehst du dorthin.

Beispiel: Zwei grüne Felder A und B stehen zur Wahl. Von A aus wären noch drei freie Felder erreichbar, von B aus nur eines. Du springst nach B. Das wirkt widersinnig, weil du dir die Wahl verengst. Es ist aber der Kern der Regel.

### Warum das funktioniert

Ein Feld mit wenigen freien Nachbarn wird bald nicht mehr erreichbar sein, wenn man es jetzt liegen lässt: Jeder Zug auf einen seiner Nachbarn nimmt ihm einen Zugang weg. Hat ein Feld nur noch einen Zugang, ist es die letzte Gelegenheit; danach wäre es nie mehr zu besuchen. Die Regel besucht also zuerst die schwer erreichbaren Felder und spart die leicht erreichbaren für später auf. Deshalb läuft der Springer zunächst am Rand und in den Ecken entlang, und die Mitte füllt sich zum Schluss von selbst. Du siehst das im Spiel: Die roten Felder bilden erst einen Ring und schließen sich dann nach innen.

### Grenzen

- Es ist eine **Heuristik**, kein Beweis. Auf dem 8x8-Brett führt sie in der Regel zum Ziel, eine Garantie gibt es nicht. Bei Gleichstand kann die falsche Wahl in der Sackgasse enden.
- Eine verbesserte Gleichstandsregel veröffentlichte Ira Pohl 1967: Bei Gleichstand zählt man die Folgezüge der Nachfolger, also einen Schritt weiter. Das kostet mehr Rechnerei.
- Mit Warnsdorff entsteht nicht unbedingt eine **geschlossene** Tour, bei der der letzte Sprung wieder auf das Startfeld führt. Das Spiel verlangt das auch nicht.

### Warum das Programm sie nicht selbst anwendet

Die grünen Hinweise zeigen die möglichen Ziele. Die Warnsdorff-Zahl dazu, also die freien Folgezüge jedes Ziels, rechnet das Programm nicht aus. Das wäre mit 256 Schritten kaum noch unterzubringen. Du machst die Zählung selbst, das Programm hält nur den Zustand und zeigt die Felder. Beim Testen des Programms habe ich genau diese Regel von einer Testschleife ausführen lassen. Sie hat damit eine vollständige Tour mit 64 Feldern erreicht.

## Geschichte des Rösselsprungs

- Das Problem ist alt und wurde schon lange vor Euler untersucht. Die genauen frühen Quellen habe ich nicht geprüft.
- **Leonhard Euler** analysierte es als einer der ersten systematisch. Er nannte es eine „kuriose Frage, die sich keiner Analyse fügt“, und entwickelte dann doch ein Verfahren, mit dem sich beliebig viele Touren bilden lassen. Die Frage war ihm an einem Schachabend gestellt worden, so erzählt er es selbst.
- **H. C. von Warnsdorff** veröffentlichte 1823 in Schmalkalden seine Regel unter dem Titel *Des Rösselsprunges einfachste und allgemeinste Lösung*.
- Wie viele Touren es gibt, wurde erst mit Computern beantwortet. Brendan McKay zählte 1997 die geschlossenen Touren auf dem 8x8-Brett: 13.267.364.410.532, wenn man Start und Richtung nicht unterscheidet. Eine kurz zuvor veröffentlichte Zahl von Martin Löbbing und Ingo Wegener (1996) war nicht durch 4 teilbar und damit falsch. Für die offenen Touren nannte Alexander Chernov 2014 etwa 1,96 · 10^16 (ob die Zahl stimmt, ist unklar).
- Allen Schwenk bewies 1991, für welche Bretter eine geschlossene Tour existiert. Bretter mit zwei ungeraden Seiten haben keine, ebenso Bretter mit einer Seite 1, 2 oder 4.
- Der Schriftsteller **Georges Perec** legte seinem Roman *Das Leben. Gebrauchsanweisung* einen Rösselsprung über ein 10x10-Raster zugrunde, die Kapitel folgen den Zügen. Perec gehörte zur Gruppe Oulipo, die Literatur mit mathematischen Zwängen schrieb.

Quellen: Wikipedia „Knight's tour“, plus.maths.org „Knight's tour“. Die frühen Quellen vor Euler habe ich nicht selbst geprüft.

## Wie es gemacht ist

Der 2090 hat keine indirekte Adressierung, rechnet nur mit 0-15 und hat nur ein Rücksprungregister. Springer passt trotzdem in 211 Schritte. Schreibweise wie oben: `RF` ist ein Register, `#F` ein Wert.

### Position in Registernummer und Maske

`POS` rechnet X und Y in die Brettregisternummer `P` = 2·Y + (X ≥ 4) und die Maske `K` = 1 << (X mod 4) um. Das Aufdoppeln für die Zeile ist ein `ADD RE,RE`, das Ergebnis von X ≥ 4 holt `CMPI` über den Carry, und `ADC` addiert ihn dazu. Die Maske entsteht ohne Schleife: X mod 4 + 1 ist `#1`, `#2`, `#3` oder `#4`. Für `#1` und `#2` stimmt der Wert schon (die Masken `#1`, `#2`), für `#3` und `#4` wird `#2` abgezogen und zweimal verdoppelt (`#1` → `#4`, `#2` → `#8`).

```
POS:  MOV  R9,RE       ; RE = Y
      ADD  RE,RE       ; RE = 2 * Y
      CMPI #3,R8       ; Carry = 1, wenn X > #3, also X >= 4
      ADC  RE          ; RE = 2 * Y + Carry  = P
      MOV  RE,R1       ; R1 = P
      MOV  R8,R2       ; R2 = X
      ANDI #3,R2       ; R2 = X mod 4        (#0 ... #3)
      ADDI #1,R2       ; R2 = #1 ... #4
      CMPI #2,R2       ; Carry, wenn R2 > #2
      BRC  PSH
      RET              ; #1 und #2 sind schon die Masken
PSH:  SUBI #2,R2       ; #3 -> #1, #4 -> #2
      ADD  R2,R2       ; verdoppeln
      ADD  R2,R2       ; noch einmal: #4 oder #8
      RET
```

### Setzen und Lesen mit einer Routine

`ZUGR` addiert den Wert in `RF` zum Brettregister `M[P]` und gibt den neuen Wert in `RF` zurück:

- **`RF` = `K`:** Feld als besucht eintragen,
- **`RF` = `#0`:** nur lesen. Das anschließende `AND R2,RF` zeigt, ob das Feld schon besucht ist.

Die Bits überlappen sich nie, weil nur auf ein freies Feld gesetzt wird. Deshalb gibt es keinen Übertrag.

```
      CALL POS         ; R1 = P, R2 = K des Zielfeldes
      MOVI #0,RF       ; nur lesen
      CALL ZUGR        ; RF = Inhalt von M[P]
      AND  R2,RF       ; RF = RF UND K
      BRZ  FREE        ; #0: Feld ist frei
```

### Zwei Brett-Hälften, ein Fenster

`R0`-`R7` sind das Fenster auf das Brett. Die untere Hälfte (`M0`-`M7`, Y < 4) kommt mit `EXRL` hinein, die obere (`M8`-`MF`, Y ≥ 4) mit `EXRA, EXRM, EXRA`. Ob die obere Hälfte gebraucht wird, steht im Register `R9`: Y ≥ 4 (`CMPI #3,R9`). So braucht das Programm für das Fenster kein eigenes Merkregister, und dieselbe Folge schließt es wieder.

```
ZUGR: CMPI #3,R9       ; Carry = 1, wenn Y > #3
      BRC  ZUP
      EXRL             ; untere Hälfte: R0-R7 = M0-M7
      GOTO ZSEL
ZUP:  EXRA             ; obere Hälfte: R0-R7 = M8-MF
      EXRM
      EXRA
ZSEL: ANDI #7,RE       ; welches Register im Fenster: P mod 8
      ...              ; Vergleichsbaum bis zum richtigen Register, z. B.:
Z0:   ADD  RF,R0       ; R0 = R0 + RF
      MOV  R0,RF       ; RF = neuer Wert
      GOTO ZT
ZT:   CMPI #3,R9       ; dieselbe Entscheidung noch einmal ...
      BRC  ZTU
      EXRL             ; ... und das Fenster schließen
      RET
```

### Die Tasten als Sprungkarte

Die Tasten liegen wie auf einem 4x4-Feld: k = 4·Reihe + Spalte. Reihe und Spalte `#0`, `#1`, `#2`, `#3` bedeuten −2, −1, +1, +2, und daraus entstehen dx und dy. Die Umrechnung ist ein Dreischritt ohne Fallunterscheidung: Wert = Zahl + (Zahl > 1) − 2. `CMPI` setzt den Carry für Zahl > 1, `ADC` addiert ihn, `SUBI #2` zieht 2 ab.

```
      KIN  R0          ; R0 = gedrückte Taste (#0 ... #F)
      MOV  R0,R5
      ANDI #3,R5       ; R5 = Spalte (k mod 4)
      CMPI #1,R5       ; Carry = 1, wenn Spalte > #1
      ADC  R5          ; R5 = Spalte + Carry
      SUBI #2,R5       ; R5 = dx = Spalte + Carry - 2
      MOV  R0,R6
      SHR  R6          ; zweimal: R6 = k div 4 = Reihe
      SHR  R6
      CMPI #1,R6
      ADC  R6
      SUBI #2,R6       ; R6 = dy
```

### Gültige Taste durch Parität

Welche der 16 Tasten Springertasten sind, erkennt das Programm an einem einzigen Bit. Ein Springersprung ist ±1 in einer Richtung und ±2 in der anderen. Die Summe dx + dy ist dann ungerade. Die Ecktasten (`#0` `#3` `#C` `#F`) haben ±2 und ±2, die mittleren (`#5` `#6` `#9` `#A`) haben ±1 und ±1: Die Summe ist gerade. `ANDI #1` auf die Summe sagt also, ob die Taste gültig ist.

```
      MOV  R5,R0
      ADD  R6,R0       ; R0 = dx + dy
      ANDI #1,R0       ; Bit 1 = ungerade ?
      BRZ  INVAL       ; gerade: keine Springertaste
```

### Zuggültigkeit durch eine Vorzeichen-Beobachtung

Ein Zug addiert dx und dy zu X und Y. Negative Werte erscheinen im 4-Bit-Register als `#8`-`#F`, und alles ab `#8` liegt auch über dem Brett. Ein einziger Vergleich (`CMPI #7`, Carry) fängt deshalb beide Ränder in einem Test ab, links wie rechts, oben wie unten. Danach liest das Programm das Brett: Ist das Zielfeld schon besucht, gilt der Zug nicht.

```
PROBE: MOV  R3,R8      ; Ziel = aktuelles Feld ...
       MOV  R4,R9
       ADD  R5,R8      ; ... + (dx, dy)
       ADD  R6,R9
       CMPI #7,R8      ; Carry = 1, wenn X > #7: rechts raus oder negativ (#8-#F)
       BRC  BAD
       CMPI #7,R9      ; dasselbe für Y
       BRC  BAD
```

### Ein Durchlauf, drei Aufgaben

Dieselbe Prüfung dient als Einzelprüfung für den Zug der Taste und als Durchlauf über alle acht Richtungen. Im Durchlauf entscheidet das Register `RA`, was mit einem freien Zielfeld geschieht:

- **Zählen** (nach jedem Zug): Gibt es überhaupt ein freies Zielfeld? Wenn nicht, ist das Spiel zu Ende.
- **Hinweise:** Jedes freie Zielfeld wird grün angeschaltet.

Die acht Richtungen entstehen durch Rechnen: Start mit (dx, dy) = (1, 2). Dreimal um 90° drehen, (dx, dy) → (−dy, dx), das sind vier Richtungen. Dann dx und dy vertauschen und noch dreimal drehen, das sind die anderen vier. Das Register `RB` zählt von `#F` bis `#8` und sagt, wo im Durchlauf man ist. Steht `RB` auf `#0`, ist es eine Einzelprüfung. Weil das Programm nur eine Rücksprungadresse hat, ist der Durchlauf kein Unterprogramm, sondern läuft im Hauptprogramm: Am Ende entscheidet `RA`, wohin es weitergeht.

```
NEXT: SUBI #1,RB       ; RB zählt #F, #E, ... herunter
      CMPI #7,RB
      BRZ  PEND        ; RB = #7: alle acht probiert
      CMPI #B,RB
      BRZ  SWAP        ; RB = #B: nach vier Richtungen tauschen
      MOV  R6,R0       ; drehen: (dx, dy) -> (-dy, dx)
      MOV  R5,R6       ; dy = dx
      MOVI #0,R5
      SUB  R0,R5       ; dx = 0 - alter dy
      GOTO PROBE
SWAP: MOV  R5,R0       ; dx und dy tauschen
      MOV  R6,R5
      MOV  R0,R6
      GOTO PROBE
```

### Hinweise ohne Löschen: push und pop

Bevor die grünen Punkte gezeichnet werden, legt das Programm die Bitmatrix des ESP auf dessen Stapel (ACK 13, push). Beim nächsten gültigen Zug holt ein einziger Befehl (pop) sie zurück, und alle Hinweise sind weg. Das Programm muss sich die Hinweisfelder nicht merken und sie nicht einzeln löschen. Die grüne Farbe bleibt in der Farbmatrix stehen, schadet aber nicht: Ein Feld bekommt seine Farbe jedes Mal neu, wenn es leuchtet.

```
INVAL: CMPI #0,RA      ; Hinweise schon sichtbar?
       BRC  LOOP       ; ja: nichts tun
       MOVI #1,RA      ; RA = #1
       MOVI #2,RF      ; zweites Nibble: #2 = push
       MOVI #D,RE      ; ACK 13 (RE = #D)
       CALL CMD
       ...             ; grün einstellen, Durchlauf über alle Richtungen

MOVE:  CMPI #0,RA      ; Zug: Hinweise sichtbar?
       BRZ  MV1
       MOVI #0,RA
       MOVI #6,RF      ; #6 = pop: Hinweise weg
       MOVI #D,RE      ; ACK 13
       CALL CMD
```

### Farben ohne Bildspeicher

Ein Zug braucht vier Befehle: Das alte Feld wird rot gefärbt (nur die Farbe, das Pixel bleibt an: Zeichenart `#0`), das neue gelb angeschaltet. Zählt man Farbwahl und Punkt je Feld, sind das vier Matrix-Befehle statt eines ganzen Bildes mit 16 Registern. Die Matrix-Befehle laufen alle durch dieselbe Routine `CMD`. Die Einsprünge enden mit `GOTO CMD`, nicht mit `CALL`: So kehrt deren `RET` direkt zum Aufrufer zurück.

```
      MOVI #1,RF       ; Farbe #1 = rot
      CALL COLOR
      CALL PTCOL       ; altes Feld nur umfärben
      ...              ; Ziel wird das aktuelle Feld
      MOVI #3,RF       ; Farbe #3 = gelb
      CALL COLOR
      CALL PTON        ; neues Feld gelb anschalten
```

### Zähler ohne Umrechnung

Der Zähler der besuchten Felder ist von Anfang an dezimal: `ADDI #1,RC`, bei Überlauf über `#9` wird `RC` auf `#0` gesetzt und `RD` erhöht. So braucht die Anzeige kein `HXDZ`, und 64 steht als `64` da. Das Display ist nur an, solange das Programm auf eine Taste wartet. Für Matrix-Befehle muss es aus sein, sonst stimmt der 9,3-ms-Takt nicht.

```
      ADDI #1,RC       ; Einer + 1
      CMPI #9,RC       ; Carry = 1, wenn RC > #9
      BRC  ZEHN
      GOTO PINIT
ZEHN: MOVI #0,RC       ; Einer zurück auf #0
      ADDI #1,RD       ; Zehner + 1
      GOTO PINIT
```

### Startfeld

`RND` legt drei Ziffern in `RD`, `RE`, `RF` ab. Das Programm nimmt `RD` und `RE`, maskiert sie mit `#7` und hat damit X und Y des Startfelds. Weil `RND` die Register `RD`-`RF` überschreibt, kommt der Zufall vor allen anderen Einstellungen.

```
      RND              ; RD, RE, RF = Zufallsziffern
      MOV  RD,R8
      ANDI #7,R8       ; X = #0 ... #7
      MOV  RE,R9
      ANDI #7,R9       ; Y = #0 ... #7
```
