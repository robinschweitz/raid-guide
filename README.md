# raid-guide

Raid-Guide der Gilde Bratwurst des Schicksals (Rising-Gods, WotLK 3.3.5a): Bossabläufe, Aufgaben pro Rolle, passende Fähigkeiten pro Klasse, Klassenguides und kurze Clips.

Die verbindliche Spezifikation steht in [docs/spec-raid-guide.md](docs/spec-raid-guide.md). Der Lichkönig-Prototyp liegt unter `docs/prototype/lichkoenig.html` und lässt sich direkt im Browser öffnen.

## Aufbau

| Pfad | Inhalt |
| --- | --- |
| `packages/guide-data` | Inhalte als YAML, Schema und Prüfung |
| `packages/site` | Website |
| `packages/tools` | Extraktion der Zauberdaten aus dem Client |
| `packages/clips` | Szenendateien, Lua-Runner, Aufnahmesteuerung |
| `docs` | Spezifikation, Quellen, Prototyp, Ideen |

Die Pakete sind noch leer. Meilenstein 1 der Spezifikation füllt `guide-data` und `site`.
