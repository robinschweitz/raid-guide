# Raid-Guide: Spezifikation und Start-Prompt

Stand: 13.11.2026 · Owner: Robin

## Ziel

Eine Website, auf der jedes Mitglied der Gilde Bratwurst des Schicksals (Rising-Gods, WotLK 3.3.5a) für jeden Raidboss nachschlägt, was passiert, was die eigene Rolle tut und welche eigene Fähigkeit welche Aufgabe löst. Dazu kommt ein Guide pro Klasse und Spec. Sie ist die eine verlässliche Quelle, für Anfänger wie für Veteranen.

Erfolg zeigt sich an drei Fällen:

- Ein neuer Paladin findet in unter 30 Sekunden, womit er beim Lichkönig eine Val'kyr betäubt.
- Der Raidleiter erzeugt in unter 2 Minuten eine Stun-Einteilung für seinen Raid als Chattext.
- Jede wichtige Mechanik hat einen kurzen Clip in echter Spielgrafik, einmal richtig und einmal falsch gespielt.

Erster Umfang: Eiskronenzitadelle 25 normal, beginnend mit dem Lichkönig.

## Nicht-Ziele

Diese Punkte werden ausdrücklich nicht gebaut. Sie kosten viel Zeit und bringen dem Ziel wenig.

- Keine Kampf-Engine und keine Simulation von Kämpfen.
- Kein Charaktermodell: keine Ausrüstung, Talente, Sockel, Procs oder Schadensberechnung.
- Kein eigener 3D-Renderer und keine stilisierte Grafik. Bewegtbild kommt aus dem echten Spielclient.
- Kein eigener Schadens-Simulator für Klassen. Zahlen zu Rotation und Ausrüstung kommen aus bestehenden Simulatoren.
- Kein Backend, keine Konten, keine Datenbank. Die Seite ist statisch.
- Keine Spieldateien von Blizzard im Repository.

## Neues Repository

Das Projekt startet in einem neuen, leeren Repository als TypeScript-Monorepo mit pnpm.

| Paket | Inhalt |
| --- | --- |
| `packages/guide-data` | Inhalte als YAML, Schema und Prüfung |
| `packages/site` | Website |
| `packages/tools` | Extraktion der Zauberdaten aus dem Client |
| `packages/clips` | Szenendateien, Lua-Runner, Aufnahmesteuerung |
| `docs` | Spezifikation, Quellen, Prototyp |

Aus raid-sim dürfen der DBC-Leser und die Fauldarm-Recherche kopiert werden. Sonst wird nichts übernommen.

## Bausteine

Fünf Bausteine: Daten, Website, Fähigkeiten-Tabelle, Clips und Klassenguides. Die Daten sind das Fundament, alles andere liest daraus.

### 1. Daten

Alle Inhalte liegen als YAML-Dateien im Repo und werden beim Build gegen ein Schema geprüft (zod).

- **Boss:** Phasen mit Kurzbeschreibung, Anweisungen pro Rolle (Tank, Heiler, Nahkampf, Fernkampf), typische Fehler.
- **Aufgabe:** gehört zu einer Phase, hat einen Zweck in einem Satz und verlangt Effekt-Typen (Stun, Slow, Unterbrechen, Magie entfernen, Krankheit entfernen, Wutanfall entfernen).
- **Fähigkeit:** Klasse, Specs, Effekt-Typen, Abklingzeit, Dauer, Reichweite, Voraussetzungen.
- **Wirkung:** Paar aus Aufgabe und Fähigkeit mit Status und Notiz. Sie wird aus dem Effekt-Typ abgeleitet und lässt sich pro Paar überschreiben.
- **Clip:** Datei, Mechanik, Variante (richtig oder falsch), Perspektive.

Jede spielbezogene Angabe trägt einen Status und ihre Quelle:

- `erwartet`: aus Client-Daten, AzerothCore oder Guides abgeleitet.
- `bestätigt`: auf Rising-Gods getestet, mit Datum und Tester.
- `wirkt nicht`: mit Begründung.

Beobachtungen von Rising-Gods ersetzen den Ausgangswert nicht stillschweigend. Beide bleiben sichtbar.

### 2. Website

Statische Seite mit Astro, interaktive Teile als React-Inseln, zuerst für Handybreite gebaut.

- **Bossseite:** Kurzfassung pro Rolle, Ablauf nach Phasen, Mechaniken mit Clips, typische Fehler, Detailwerte aufklappbar.
- **Meine Aufgaben:** Klasse und Spec wählen. Die Seite zeigt alle Aufgaben des Bosses mit passenden Fähigkeiten, nicht wirkenden Fähigkeiten und einer Alternative für den Fall, dass man nichts Passendes hat.
- **Einteilung:** Raid eingeben, Aufgabe wählen. Die Seite zeigt, wer es kann, und erzeugt einen kopierbaren Chattext. Bei Aufgaben mit mehreren Zielen und Wellen wird nach Abklingzeit verteilt.
- **Suche** über alle Inhalte.

Der vorhandene HTML-Prototyp zum Lichkönig ist Vorlage für Aufbau, Texte und Startdaten.

### 3. Fähigkeiten-Tabelle aus Client-Daten

Ein Extraktionswerkzeug liest für alle Spielerzauber Mechanik und Aura-Typen aus `Spell.dbc` und schlägt Effekt-Typen vor. Immunitäten der Gegner kommen als Vorbelegung aus `creature_template` von AzerothCore. Das Ergebnis ist eine Vorschlagsdatei, die ein Mensch durchsieht. Nichts wird ohne Durchsicht veröffentlicht.

### 4. Clips

Clips zeigen echtes Spielbild aus dem 3.3.5a-Client und entstehen kontrolliert auf einem lokalen Testserver (AzerothCore mit Eluna).

1. Eine Szenendatei pro Clip legt fest: Boss, Mechanik, Variante, Positionen der Statisten, auszulösender Zauber mit Ziel, Kamera, Dauer.
2. Ein Lua-Runner baut die Szene auf, schaltet die Boss-KI ab und lässt den Boss den echten Zauber wirken.
3. Ein Steuerskript startet und stoppt OBS pro Szene über dessen WebSocket-Schnittstelle und benennt die Datei.
4. ffmpeg erzeugt das Endformat: 10 bis 20 Sekunden, 720p, ohne Ton, höchstens 3 MB.
5. Der Owner sieht jeden Clip an, bevor er veröffentlicht wird.

Mechaniken, die am Boss-Skript hängen und sich nicht einzeln auslösen lassen, werden im echten Kampf mit GM-Unverwundbarkeit aufgenommen. Clips liegen außerhalb von Git.

### 5. Klassenguides

Jede Spec bekommt eine Seite nach demselben Aufbau. Sie erklärt, wie man die Spec im Raid spielt, und verknüpft sie mit den Bossen.

- **Kurzfassung:** Rolle im Raid und die drei wichtigsten Regeln für Einsteiger.
- **Spielweise:** Rotation oder Prioritätenliste, Einsatz der Cooldowns, Besonderheiten bei Bewegung und mehreren Zielen.
- **Aufbau des Charakters:** Talente, Glyphen, Reihenfolge der Werte, Sockel, Verzauberungen, Verbrauchsgüter.
- **Nutzen für den Raid:** Buffs, Debuffs und alle Fähigkeiten aus der Fähigkeiten-Tabelle (Stun, Slow, Unterbrechen, Bannen).
- **Bei welchem Boss brauche ich was:** wird automatisch aus den Aufgaben der Bosse erzeugt und nicht von Hand geschrieben.
- **Typische Fehler** und wie man sie erkennt.

Die Inhalte liegen als Daten und Text im selben Schema wie die Bosse. Jede Angabe trägt Quelle und Status. Zahlen zu Rotation und Ausrüstung stammen aus wowsims oder einem etablierten Guide und werden nicht von der KI geschätzt. Pro Spec benennt der Owner einen erfahrenen Spieler der Gilde, der die Seite vor der Veröffentlichung prüft.

Die Reihenfolge der Specs richtet sich danach, was in der Gilde gespielt wird. Die erste Spec dient als Vorlage für alle weiteren.

### Später, nicht jetzt

- Frage-Bot im Discord, der nur aus diesen Daten antwortet.
- Dieselben Szenen als spielbare Übung auf einem Gilden-Testserver.
- Plan-Simulator: Der Raidleiter ändert eine Einteilung und sieht das Ergebnis.

## Meilensteine

Neun Schritte, jeder endet mit einer Vorführung. Der nächste beginnt erst nach Freigabe durch den Owner.

1. **Lichkönig-Seite.** Datenschema, Lichkönig 25 normal vollständig als Daten (aus dem Prototyp übernommen), Bossseite und Meine Aufgaben.
   - Abnahme: läuft lokal und auf dem Handy, beide Ansichten sind vollständig, die Schema-Prüfung ist grün.
2. **Einteilung.** Raid-Eingabe und Vorschlag pro Aufgabe.
   - Abnahme: Stun-Einteilung für einen 25er-Beispielraid in unter 2 Minuten, Chattext kopierbar.
3. **Fähigkeiten-Vorschläge.** Extraktion für alle zehn Klassen.
   - Abnahme: Die Vorschlagsdatei liegt vor. Für den Lichkönig sind die Abweichungen zu den Handdaten aufgelistet.
4. **Erster Klassenguide.** Eine Spec vollständig, als Vorlage für alle weiteren. Der Owner wählt die Spec.
   - Abnahme: Der Prüfer der Spec hat die Seite freigegeben. Der Abschnitt „Bei welchem Boss brauche ich was“ entsteht ohne Handarbeit aus den Lichkönig-Daten.
5. **Ein Clip von Anfang bis Ende.** Entweihen, richtig und falsch, über Szenendatei, Runner, OBS und ffmpeg.
   - Abnahme: Ein Befehl erzeugt beide Clips. Sie sind auf der Bossseite eingebunden.
6. **Alle Lichkönig-Clips.**
   - Abnahme: Jede Mechanik der Seite hat mindestens einen Clip.
7. **Zweiter Boss: Fauldarm.** Nur Daten und Szenen.
   - Abnahme: Kein Code an Seite oder Werkzeugen geändert, Aufwand höchstens ein Arbeitstag.
8. **Restliche Bosse der Eiskronenzitadelle.** Je Boss wie Schritt 7.
9. **Weitere Klassenguides.** Je Spec wie Schritt 4, nur Inhalte.
   - Abnahme pro Spec: Freigabe durch den Prüfer, kein Code geändert.

## Arbeitsregeln für die KI

Diese Regeln gelten für jede Sitzung und gehen jedem eigenen Vorschlag der KI vor.

- **Sichtbares Ergebnis:** Nach jedem Meilenstein gibt es etwas, das der Owner im Browser ansehen kann.
- **Kurze Pläne:** Ein Plan pro Meilenstein ist höchstens eine Seite lang und enthält keinen Code.
- **Der einfachste Weg gewinnt:** Keine neue Abstraktion, bevor sie zweimal gebraucht wurde.
- **Kein zusätzlicher Umfang:** Ideen außerhalb dieser Spezifikation kommen als ein Satz in `docs/ideas.md` und werden nicht gebaut.
- **Spielwissen wird nie erfunden:** Was nicht belegt ist, bekommt den Status `erwartet` mit Quelle oder wird als offene Frage notiert.
- **Rising-Gods ist die Wahrheit:** AzerothCore liefert nur die Vorbelegung.
- **Wenige Tests:** nur für die Schema-Prüfung, die Ableitung der Wirkungen und die Verteilungslogik der Einteilung.
- **Sprache:** Texte der Seite sind Deutsch, kurz und in Handlungsform, zum Beispiel „Lauf sofort raus“.
- **Fragen statt Annahmen:** Bei Unklarheit eine Frage an den Owner stellen.

## Offene Entscheidungen

Diese Punkte entscheidet der Owner. Die rechte Spalte ist ein Vorschlag.

| Frage | Vorschlag |
| --- | --- |
| Sprache der Inhalte | Deutsch |
| Lizenz | GPL-2.0, falls Code aus raid-sim oder AzerothCore kopiert wird |
| Wer sieht die Seite | Seite öffentlich, Testserver nur privat |
| Ablage der Clips | Verzeichnis auf dem nginx der Gilde |
| Wer bestätigt Angaben auf Rising-Gods | Der Raidleiter trägt Tests nach dem Raid ein |
| Stand von AzerothCore und Eluna für den Testserver | Aktueller Stand beim Aufsetzen, in `docs/sources.md` festgehalten |
| Raid-Import aus der Anmeldeseite raid-website | Später, nach Meilenstein 2 |

## Start-Prompt

```
Lies docs/spec-raid-guide.md vollständig. Das ist die verbindliche Spezifikation.

Arbeite nur an Meilenstein 1.

Vorlage für Aufbau, Texte und Startdaten ist docs/prototype/lichkoenig.html.

Schreib zuerst einen Plan von höchstens einer Seite, ohne Code, und warte auf
meine Freigabe. Stell mir vorher alle Fragen, die du zum Umfang hast.

Wenn du fertig bist, nenn mir den Befehl, mit dem ich die Seite lokal starte,
und sag mir, was ich prüfen soll.
```

Für jeden weiteren Meilenstein änderst du nur die Nummer des Meilensteins.
