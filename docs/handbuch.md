# Recscribe: Handbuch

Aufnahmen sichern, lokal transkribieren (mit Sprechertrennung), Sprecher benennen und mit einem
eigenen KI-Schlüssel zu Protokollen oder Meeting Notes verarbeiten. Kein Cloud-Account, keine
App, kein Bluetooth. Der Rekorder bleibt unverändert, solange du das Entfernen nicht
ausdrücklich einschaltest oder verlangst. Kein Audio verlässt den Rechner (nur
der Text des Transkripts, und nur wenn du die KI-Weiterverarbeitung nutzt).

Unterstützte Quelle heute: USB-Rekorder mit `.ops`-Dateien, getestet am AIREC (Jieli-SoC,
64 GB, USB-Massenspeicher). Ein allgemeiner Import beliebiger Aufnahmen (WAV, MP3, M4A, Ogg,
Sprachmemos) ist geplant. Transkription, Sprecher und KI-Teil sind davon unabhängig und
arbeiten auf den Dateien im Zielordner.

## Installation

Fertige Pakete für macOS, Linux und Windows stehen unter
[Releases](https://github.com/MatthiasHertel80/recscribe/releases). In jedem Paket liegt eine
`README.txt` mit den Schritten für das jeweilige System, auch zu den Warnungen von Gatekeeper und
SmartScreen (die Programme sind nicht signiert). Für die Transkription braucht es zusätzlich `uv`
und `ffmpeg`, siehe Abschnitt Transkription.

Das Programm heißt `recscribe`. Unter macOS liegt es in der App:
`Recscribe.app/Contents/MacOS/recscribe`. Die Beispiele unten laufen im Terminal, unter Windows in
PowerShell mit `.\recscribe.exe`.

| System | Stand |
|---|---|
| macOS (Apple Silicon, Intel) | am besten erprobt; Spracherkennung mit MLX nur auf Apple Silicon |
| Linux (x86_64, aarch64) | automatische Tests laufen; das Fenster braucht Vulkan oder OpenGL |
| Windows 10/11 (x86_64) | Beta: Rekorder als Laufwerksbuchstabe; Entfernen vom Rekorder nur endgültig (kein Papierkorb für Wechseldatenträger) |

## Benutzung

```sh
recscribe detect             # erkannten Rekorder anzeigen
recscribe status             # Gerät gegen Sync-Stand vergleichen
recscribe sync --dry-run     # nur anzeigen
recscribe sync               # neue Aufnahmen kopieren
recscribe gui                # Fenster öffnen (alles unten per Maus)
recscribe report --open      # HTML-Übersicht öffnen
recscribe convert            # .ops verlustfrei nach .opus umpacken (--wav: auch 16-kHz-WAV)
recscribe transcribe         # lokal transkribieren, mit Sprechertrennung
recscribe install-agent      # Sync automatisch beim Anstecken (launchd, macOS)
recscribe uninstall-agent
recscribe install-app        # App "Recscribe" mit Icon nach /Applications (macOS)
recscribe uninstall-app
```

Optionen für alle Befehle (oder per Umgebungsvariable):

| Option | Variable | Standard |
|---|---|---|
| `--dest DIR` | `RECSCRIBE_DEST` | `~/Aufnahmen/Recscribe` |
| `--source DIR` | `RECSCRIBE_SOURCE` | automatische Erkennung |

`sync --settle-secs N` (Standard 2) wartet kurz und überspringt Dateien, die danach größer
sind, also noch aufgenommen werden. `sync --notify` zeigt eine macOS-Benachrichtigung.
`sync --progress` gibt `[i/n] Name` und einen überschreibenden Byte-Zähler aus (das Fenster nutzt
das für seinen Fortschrittsbalken). Eine Sperre, die ein abgebrochener Sync hinterlassen hat,
wird beim nächsten Lauf übernommen, sobald es den Prozess nicht mehr gibt.

## Vom Rekorder entfernen

Standard ist: Der Rekorder wird nur gelesen. Entfernen ist ausdrücklich zu wünschen, auf drei
Wegen:

```sh
recscribe sync --delete-after             # nach dem Kopieren, nur für diesen Lauf
recscribe settings --delete-after-sync on # dauerhaft für jeden Sync, auch den des Agenten
recscribe delete 20260804110159           # gezielt eine Aufnahme (oder mehrere)
recscribe delete --all                    # alle, die sicher gesichert sind
```

`--dry-run` zeigt vorher, was passieren würde. `--keep` schlägt die Einstellung für einen Lauf.
`delete` fragt im Terminal nach, ohne Terminal braucht es `--yes`. Im Fenster stehen dafür in der
Übersicht ein Schalter, die Art des Entfernens, ein Knopf für alle gesicherten Aufnahmen und pro
Aufnahme ein Knopf, jeweils mit Rückfrage.

**Wann eine Aufnahme weg darf.** Nur wenn alles davon stimmt:

- Das Manifest kennt sie, und ihre Größe passt zur Datei auf dem Rekorder.
- Die lokale Kopie in der Bibliothek ist vorhanden und gleich groß.
- Bei Aufnahmen, die nicht eben erst in diesem Lauf kopiert wurden, werden Kopie und Rekorderdatei
  neu gehasht und müssen zur gespeicherten SHA-256 passen. Frisch kopierte wurden beim Kopieren
  schon so geprüft.
- Die Quelle ist nicht die lokale Kopie selbst und liegt nicht im Zielordner. Wer `--source` in
  die Bibliothek zeigen lässt, kann damit nichts löschen.
- Direkt vor dem Entfernen hat sich die Größe nicht geändert (die Aufnahme wächst nicht gerade).

Was die Prüfung nicht besteht, bleibt auf dem Rekorder, mit Grund in der Ausgabe. Nur `.ops`-Dateien
im RECORD-Ordner kommen in Frage. Eine unlesbare Einstellungsdatei bedeutet: nichts entfernen.

**Wohin.** Standard ist der Papierkorb des Datenträgers, rückholbar (`--method trash`): auf macOS
`<Rekorder>/.Trashes/<uid>/`, das der Finder im Papierkorb zeigt, sonst `.Trash-<uid>` nach
freedesktop.org. Das gibt auf dem Rekorder erst Platz frei, wenn du den Papierkorb leerst.
`--method permanent` löscht sofort und endgültig. Windows hat für Wechseldatenträger keinen
Papierkorb: Dort gibt es nur `permanent` (das ist dort auch der Standard), `--method trash`
meldet "nicht unterstützt" und entfernt nichts. Die Art stellst du auch mit
`recscribe settings --delete-method permanent` oder im Fenster ein.

Auf macOS mit einem echten FAT32-Datenträger geprüft: Die Dateien landen in `.Trashes/<uid>/`
(zusammen mit den `._`-Begleitdateien, die macOS auf FAT anlegt). Nicht erprobt ist, wie die
Firmware des Rekorders danach reagiert: Sie bekommt nur normale FAT-Änderungen mit, geprüft wurde
das mangels Gerät nicht.

## Als App (macOS)

`recscribe install-app` legt `Recscribe.app` mit Icon nach `/Applications` (ohne Schreibrecht
dort nach `~/Applications`, anderer Ort mit `--dir`). Die App enthält eine Kopie des Programms;
ein Doppelklick öffnet das Fenster, ein Start ohne Befehl tut dasselbe. Nach einem Update den
Befehl noch einmal ausführen. `recscribe uninstall-app` entfernt sie wieder,
fremde Programme gleichen Namens bleiben unberührt. Die App ist nur ad hoc signiert, sie läuft
also auf diesem Rechner, ist aber nicht für die Weitergabe gedacht. Beim ersten Zugriff auf den
Rekorder fragt macOS nach dem Zugriff auf Wechseldatenträger.

## Fenster

`recscribe gui` öffnet ein natives Fenster (egui, auf macOS mit Metal). Es ruft dieselben
Befehle auf wie das Terminal, es gibt also keine zweite Logik.

- **Übersicht:** Kennzahlen (Aufnahmen, noch zu holen, transkribiert, ohne Sprechernamen,
  Gesamtdauer), Rekorderstatus und eine Liste aller Aufnahmen. Knopf "Jetzt synchronisieren".
  Die Liste aktualisiert sich alle 10 s, damit das Anstecken sofort auffällt.
- **Transkripte:** Liste links, rechts der Text mit Sprecherfarben, Zeitmarken, Suche mit
  Treffermarkierung, Redeanteilen. Knöpfe: Markdown/SRT öffnen, im Ordner zeigen, Text kopieren.
- **Sprecher:** Wie `recscribe speakers`, nur mit Abspielknopf je Probe und Namensfeldern. Namen
  aus früheren Aufnahmen stehen als Schnellauswahl daneben. "Speichern" schreibt das
  Transkript neu, "Namen zurücksetzen" löscht sie.
- **Transkription:** Erkennung (Auto, MLX, faster-whisper), Modell, Sprache, Starthinweis (als
  Standard speicherbar), Sprecher trennen, neu berechnen, offline. Aufnahmen per Häkchen
  wählen, Fortschrittsbalken und Protokoll unten, "Abbrechen" beendet auch uv und Python.

Sync und Transkription laufen als Kindprozess, das Fenster friert nie ein. Es läuft immer nur
ein Auftrag. Abspielen nutzt `afplay` (macOS), sonst ffplay, paplay oder aplay.

### Sprache des Fensters

Das Fenster folgt der Systemsprache: Deutsch bei deutschem System, sonst Englisch. Unter macOS
zählt die erste unterstützte Sprache aus den Spracheinstellungen, unter Windows das Gebietsschema
des Benutzers, unter Linux `LANGUAGE`, `LC_ALL`, `LC_MESSAGES`, `LANG`. Zum Überschreiben: `recscribe gui --lang en` oder
`RECSCRIBE_LANG=de`. Übersetzt ist nur die Oberfläche. Die Ausgabe des Terminals, die Protokollzeilen
im unteren Bereich des Fensters und die Dateien (Transkripte, Standardname "Sprecher 1") bleiben deutsch.

## KI-Weiterverarbeitung: Protokoll, Meeting Notes, Aufgaben

Aus einem Transkript macht ein Sprachmodell ein Protokoll, Meeting Notes, eine Kurzfassung, eine
Aufgabenliste oder technische Notizen. Das Ergebnis liegt als `<Aufnahme>.<Vorlage>.md` neben dem
Transkript (zum Beispiel `20260804110159.protokoll.md`) und ist reiner Markdown-Text.

**Eigener Schlüssel (BYOK).** Es gibt keinen Dienst von uns dazwischen, das Programm spricht direkt
mit dem Anbieter deiner Wahl:

| Anbieter | Einstellung | Hinweis |
|---|---|---|
| Anthropic (Claude) | `--provider anthropic` | Standardmodell `claude-sonnet-5-5`, mit `--model` änderbar |
| OpenAI-kompatibel | `--provider openai --model NAME` | OpenAI, OpenRouter, und lokal Ollama (`--base-url http://localhost:11434/v1`) oder LM Studio |

Bei einem lokalen Dienst bleibt das Transkript auf dem Rechner, ein Schlüssel ist dort nicht nötig.
Sonst geht der gesamte Text des Transkripts an den Anbieter (die Ansicht zeigt den Zielrechner an).
`--dry-run --print-prompt` zeigt vorher genau, was gesendet würde.

**Wo der Schlüssel liegt.** Reihenfolge beim Lesen: Umgebungsvariable (`ANTHROPIC_API_KEY`,
`OPENAI_API_KEY` oder eine eigene per `--key-env`), macOS-Schlüsselbund, sonst eine Datei
`~/.config/recscribe/keys/<anbieter>.key` mit Rechten 0600 (Windows: `%APPDATA%\recscribe\keys`, im
Profil des Benutzers). Nie im Zielordner (der kann in einer Cloud
liegen) und nie in `ai.json`. Eine Umgebungsvariable aus dem Terminal sieht ein Fenster, das du über
den Finder startest, nicht: dort den Schlüssel im Fenster speichern. Ohne TLS (http) sendet das Programm den Schlüssel
nur an den eigenen Rechner (localhost), sonst bricht es ab.
Beim Ablegen im Schlüsselbund übergibt `security` das Geheimnis als Argument. Es ist deshalb für
Sekundenbruchteile in der Prozessliste desselben Benutzers sichtbar.

```bash
recscribe ai config --provider anthropic --language de        # Sprache der Ausgabe: auto, de, en, fr, es, it
recscribe ai config --key-stdin                               # Schlüssel eingeben (unsichtbar) und ablegen
recscribe ai test                                             # kurze Anfrage, prüft Schlüssel und Modell
recscribe ai templates                                        # Vorlagen anzeigen
recscribe ai run 20260804110159 --template protokoll --notes "Projekt Orion, Wochenmeeting"
recscribe ai run --template meeting-notes                     # alle Transkripte ohne dieses Ergebnis
recscribe ai run 20260804110159 --dry-run --print-prompt      # nichts senden, Prompt ansehen
```

**Vorlagen.** Fest eingebaut sind `protokoll`, `meeting-notes`, `zusammenfassung`, `aufgaben` (nur
Aufgaben und Beschlüsse mit Zeitmarke) und `technik` (Entscheidungen, Zahlen, Probleme, Glossar). Eigene
Vorlagen sind Textdateien in `<Ziel>/.recscribe/templates/<kennung>.md`; ein Prompt, optional mit
Kopfzeilen:

```
---
name: Wochenstatus
description: Kurz und knapp für die Teamrunde
---
Schreibe einen Wochenstatus aus dem Transkript. {{notes}}
```

Platzhalter: `{{title}}`, `{{date}}`, `{{duration}}`, `{{speakers}}`, `{{notes}}`, `{{transcript}}`. Fehlt
`{{transcript}}`, hängt das Programm das Transkript unten an. Ein unbekannter Platzhalter ist ein Fehler.
`recscribe ai templates --copy protokoll --as mein-protokoll` legt eine Kopie zum Anpassen an. Eine
eigene Datei mit der Kennung einer festen Vorlage ersetzt diese, `--delete` stellt sie wieder her.

**Schutz vor Prompt-Injection.** Das Transkript steht in `<transcript>`-Tags und der feste Systemprompt
sagt dem Modell, dass darin Daten stehen und keine Anweisungen. Er verlangt außerdem, nichts zu
erfinden (Namen, Zahlen, Termine) und unsichere Stellen mit "(?)" zu markieren. Ein Modell hält sich
nicht garantiert daran: Protokolle vor dem Verteilen lesen, vor allem Aufgaben, Zahlen und Termine.

Im Fenster ist das die Ansicht **Weiterverarbeitung**: Vorlage wählen, optional Kontext eintragen,
Aufnahmen ankreuzen, "Erzeugen". Darunter das Ergebnis mit Kopieren und Öffnen. Vorlagen lassen sich dort
kopieren, bearbeiten und löschen; die Verbindung (Anbieter, Modell, Adresse, Schlüssel, Test) klappt
darunter auf. Auch der Transkriptleser hat dafür den Knopf "Protokoll erzeugen".

Grenzen: Sehr lange Aufnahmen gehen in einer einzigen Anfrage durch (Schutzgrenze 150000 geschätzte
Tokens, `--max-input-tokens`); ein Aufteilen in Abschnitte gibt es nicht. Die Antwort wird nicht
gestreamt. Wird sie wegen `--max-tokens` (Anthropic, Standard 8192) abgeschnitten, steht das in der
Herkunftszeile und als Warnung im Protokoll.

## Ablauf und Garantien

- Erkennung über die Struktur, nicht den Namen (das Laufwerk heißt nur "NO NAME"):
  Ordner `RECORD` plus `SET.bin` im Wurzelverzeichnis oder vorhandene `.ops`-Dateien.
- Ablage als `<Ziel>/JJJJ/MM/<Name>.ops`. Neue Dateien werden in `*.part` geschrieben,
  per SHA-256 zurückgelesen und erst dann atomar umbenannt.
- `<Ziel>/.recscribe/manifest.json` hält Größe, Prüfsumme und Zeitpunkt je Datei.
  Es wird nach jeder Datei gespeichert, ein Abbruch verliert nichts.
- Ändert sich eine Datei auf dem Gerät (andere Größe), wird sie aktualisiert.
  Fehlt die lokale Kopie, wird sie neu geholt.
- Auf dem Rekorder wird nichts gelöscht, außer du verlangst es (siehe "Vom Rekorder entfernen").
- Ein Lauf schreibt zusätzlich `<Ziel>/index.html` (Kennzahlen, Minuten pro Tag, Liste).

## Das .ops-Format

Es braucht keinen eigenen Decoder. Eine `.ops`-Datei ist eine Folge von 80-Byte-Slots ohne
Header, jeder Slot ist ein normales Opus-Paket (SILK Wideband, 20 ms, mono, CBR 32 kbit/s,
auf 80 Byte aufgefüllt). `convert` packt diese Pakete unverändert in einen Ogg-Container.
Es wird nichts neu kodiert, die Qualität bleibt also exakt die des Rekorders.

- Alle Dateigrößen auf dem Gerät sind Vielfache von 64 Byte, deshalb ist der letzte Slot
  meist angeschnitten und wird verworfen (höchstens 20 ms).
- Defekte Slots werden durch einen Paketverlust-Marker ersetzt, die Zeitachse bleibt stimmig.
  `convert` meldet die Anzahl. Eine Datei, die überwiegend aus Nicht-Opus-Slots besteht,
  wird abgelehnt.
- Auch das BLE-Format (82-Byte-Sätze mit `[P`-Präfix) wird erkannt.
- Geprüft an einer echten Aufnahme (306 Slots, alle gültig): ffmpeg dekodiert ohne Meldung,
  6,12 s wie erwartet, Pegelverlauf wie Sprache.

## Transkription

```sh
recscribe transcribe                          # alle synchronisierten ohne Transkript
recscribe transcribe 20260804110159 --language de --names "Matthias,Anna"
recscribe transcribe --prompt "Cortex-M, CMSIS, Keil, TrustZone, Ethos-U"
recscribe transcribe --dry-run
```

Pro Aufnahme entstehen neben der `.ops`-Datei `<Name>.opus`, `<Name>.md` (Absätze mit
Zeit und Sprecher), `<Name>.srt` (Untertitel) und `<Name>.json`.

Ablauf: Umpacken, ffmpeg nach 16 kHz mono, Spracherkennung (Whisper large-v3-turbo mit
Wortzeiten, bei faster-whisper mit Stille-Filter), Sprechertrennung (speakrs), Zuordnung jedes Worts zum
Sprecher mit der größten zeitlichen Überlappung, Absätze bei Sprecherwechsel oder Pause
(`--max-gap`). Die Zuordnung steckt in Rust (`src/transcript.rs`) und ist getestet.

| Baustein | Wahl | Grund |
|---|---|---|
| Spracherkennung | Whisper large-v3-turbo, auf Apple Silicon über `mlx-whisper` (Metal), sonst `faster-whisper` | Starthinweis (`--prompt`) für Fachbegriffe, robust bei Raumaufnahmen, Sprachwechsel Deutsch/Englisch |
| Sprechertrennung | `speakrs` (Rust-Port der pyannote-community-1-Pipeline, CoreML auf Apple Silicon) | Genauigkeit wie pyannote, deutlich schneller, kein Hugging-Face-Token nötig |

Die Engines sind kleine Skripte in `engines/` (mit eingebettet, werden nach
`<Ziel>/.recscribe/engines/` geschrieben). `uv` installiert beim ersten Lauf ihre
Abhängigkeiten, die Modelle werden einmal geladen (auf dem Mac rund 2,1 GB im
Hugging-Face-Cache, dazu der uv-Cache). Danach geht es mit `--offline` ganz ohne Netz.

Voraussetzungen: `uv` und `ffmpeg`. macOS: `brew install uv ffmpeg`. Windows: `winget install --id astral-sh.uv`
und `winget install --id Gyan.FFmpeg`. Linux: `curl -LsSf https://astral.sh/uv/install.sh | sh` und `sudo apt install ffmpeg`.

Hinweise zur Bedienung:

- `--language auto` (Standard) erkennt die Sprache. Mit faster-whisper je Abschnitt, mit
  mlx-whisper einmal je Aufnahme. Für überwiegend deutsche Gespräche mit englischen
  Fachbegriffen ist `--language de` meist besser.
- Sprechernamen: `--names` ordnet Namen in der Reihenfolge des ersten Auftretens zu, bequemer
  geht es mit Hörproben (nächster Abschnitt). Mit `--remerge` lässt sich das Transkript aus
  den Zwischenergebnissen im Cache neu schreiben, ohne die Erkennung zu wiederholen.
  `--force` rechnet alles neu, behält aber die per `speakers` vergebenen Namen.
- Scheitert nur die Sprechertrennung, bleibt das Transkript erhalten (ohne Sprecher) und
  die Erkennung muss nicht wiederholt werden.
- Aufnahmen unter `--min-secs` (Standard 10 s) werden übersprungen.
- Pausen und Überlappungen: Sprecherwechsel mitten im Satz und gleichzeitiges Sprechen
  sind die schwache Stelle jeder Sprechertrennung. Bei Headset-nahen Aufnahmen gut, bei
  Raumaufnahmen mit Nachhall sollte man die Zuordnung stichprobenartig prüfen.
- Alternative Erkennung: NVIDIA Parakeet TDT v3 (25 europäische Sprachen, sehr schnell)
  ist nicht eingebaut. Es kennt keinen Starthinweis für Fachbegriffe.

## Sprecher per Hörprobe benennen

```sh
recscribe speakers                     # alle transkribierten Aufnahmen ohne Namen
recscribe speakers 20260804110159      # eine bestimmte Aufnahme
```

Je Sprecher wählt das Programm bis zu drei kurze Proben (`--samples`, höchstens `--length`
Sekunden, Standard 8): möglichst lange Äußerungen ohne Überlappung mit anderen Sprechern,
über die Aufnahme verteilt. Sie werden aus der Aufnahme geschnitten, mit `afplay` (macOS,
sonst ffplay, paplay oder aplay) abgespielt und mit Zeitstempel und Text angezeigt. Pro Probe:

- Name eintippen: gilt für diesen Sprecher, weiter zum nächsten
- Zahl: nimmt einen schon bekannten Namen aus einer früheren Aufnahme (wird angezeigt)
- Enter: nächste Probe desselben Sprechers, `r`: nochmal abspielen, `s`: Sprecher überspringen
- Strg+D: abbrechen, bereits vergebene Namen bleiben erhalten

Danach werden `.md`, `.srt` und `.json` neu geschrieben, ohne Erkennung zu wiederholen. Die
Namen stehen unter `<Ziel>/.recscribe/speakers/<Aufnahme>.json` (Rohlabel zu Name) und
überleben `transcribe --force`. Weitere Optionen:

- `--list`: Sprecher, Redezeit und Proben anzeigen, nichts fragen (Probedateien bleiben im Cache)
- `--name 1=Matthias --name 2=Anna`: direkt setzen, ohne Rückfrage (Nummer in Reihenfolge des
  ersten Auftretens, leerer Name löscht)
- `--reset`: Namen einer Aufnahme verwerfen
- `--no-play`: nur Text zeigen, nicht abspielen

Die Namen gelten je Aufnahme. Eine automatische Wiedererkennung derselben Stimme über mehrere
Aufnahmen hinweg gibt es nicht; bekannte Namen werden nur als Auswahl angeboten.

## Hinweise

- Die FAT-Zeitstempel des Rekorders sind unbrauchbar (Jahr 2047). Die Aufnahmezeit steht
  im Dateinamen und wird von dort gelesen.
- Die Dauer in `status` und im Report ist eine Schätzung: Dateigröße / 4000 Byte pro Sekunde.
  `convert` kennt die genaue Dauer (Pakete zählen).
- Beim ersten Zugriff fragt macOS ggf. nach Zugriff auf Wechseldatenträger. Für den
  launchd-Agent einmal `recscribe sync` im Terminal ausführen und die Abfrage bestätigen.
- Der Agent startet bei jedem Mount und beendet sich still, wenn kein Rekorder dabei ist.
  Parallele Läufe verhindert eine Sperrdatei.

## Geplant

- **Allgemeiner Import** beliebiger Aufnahmen (WAV, MP3, M4A, Ogg, Sprachmemos, Ordner
  überwachen) als zweite Quelle neben dem USB-Rekorder, mit derselben Ablage und Pipeline.
- Word-Export des Protokolls (`.docx`).
- Lange Aufnahmen für die KI in Abschnitte teilen statt einer einzigen Anfrage.
- Stimmen über mehrere Aufnahmen hinweg wiedererkennen.

