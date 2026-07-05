# MantisBT-Plugin für Claude Code

*[English version](README.md)*

Ein [Claude Code](https://claude.com/product/claude-code)-Plugin, das den [MantisBT-MCP-Server](https://github.com/dpesch/mantisbt-mcp-server) in Claude Code integriert – damit lassen sich MantisBT-Tickets direkt aus einer Claude-Code-Sitzung recherchieren, anlegen und verwalten.

## Inhalt

- **MCP-Server** — startet `@dpesch/mantisbt-mcp-server` per `npx` und verbindet sich mit der REST-API deiner MantisBT-Instanz
- **Skills**
  - `/mantisbt:research` — Recherche zu Issues, Bugs und Projekt-Historie (nur lesend)
  - `/mantisbt:create` — geführtes Anlegen eines neuen Tickets (Bug, Feature, Task)
  - `/mantisbt:sync` — Status des MantisBT-Metadaten-Caches prüfen oder aktualisieren
  - `/mantisbt:setup` — geführte Einrichtung, Status-Check und Troubleshooting für die optionale semantische Suche
- **Agent**
  - `mantis-researcher` — ein read-only Sub-Agent für token-intensive Recherchen (Bulk-Abfragen, projektübergreifende Suchen, große Ergebnismengen), isoliert vom Hauptkontext

## Voraussetzungen

- Eine MantisBT-Instanz mit aktivierter REST-API
- Ein MantisBT-API-Token (**MantisBT → Profile → API Tokens**)
- Node.js / `npx` auf dem Rechner, auf dem Claude Code läuft (wird zum Start des MCP-Servers benötigt)

## Installation

Plugin in Claude Code installieren und aktivieren:

```
/plugin install <marketplace-or-repo-reference>
```

Bei der ersten Nutzung fragt Claude Code zwei Einstellungen ab (`userConfig`):

| Einstellung | Beschreibung |
|---|---|
| **MantisBT API URL** | Basis-URL deiner MantisBT-REST-API, z.B. `https://bugs.example.com/api/rest` |
| **API Key** | Dein MantisBT-API-Token (wird sicher gespeichert, nie im Klartext angezeigt) |

## Optional: lokale semantische Suche

Der zugrunde liegende MCP-Server kann optional einen lokalen, offline laufenden semantischen Such-Index über deine MantisBT-Tickets aufbauen, damit `/mantisbt:research` Anfragen in natürlicher Sprache beantworten kann (z.B. »Login schlägt nach Passwort-Reset fehl«) statt sich auf exakte Stichwort-Treffer zu verlassen. Das Embedding-Modell (~80 MB) wird bei der ersten Nutzung einmalig heruntergeladen und läuft danach vollständig offline — ohne externe API-Aufrufe.

Standardmäßig ist die Funktion deaktiviert. Zum Aktivieren müssen folgende Umgebungsvariablen für den Prozess gesetzt werden, der Claude Code startet (sie sind nicht Teil der `userConfig` dieses Plugins, da sie den MCP-Server selbst konfigurieren):

| Variable | Standard | Funktion |
|---|---|---|
| `MANTIS_SEARCH_ENABLED` | `false` | Auf `true` setzen, um semantische Suche zu aktivieren |
| `MANTIS_SEARCH_BACKEND` | `vectra` | `vectra` (reines JS, keine zusätzliche Installation) oder `sqlite-vec` (schneller ab 10.000+ Tickets, erfordert `npm install sqlite-vec better-sqlite3`) |
| `MANTIS_SEARCH_DIR` | `{MANTIS_CACHE_DIR}/search` | Speicherort des Such-Index |
| `MANTIS_SEARCH_MODEL` | `Xenova/paraphrase-multilingual-MiniLM-L12-v2` | Name des Embedding-Modells |
| `MANTIS_SEARCH_THREADS` | `1` | Anzahl ONNX-Threads bei der Indexierung |

Details siehe [README des MantisBT-MCP-Servers](https://codeberg.org/dpesch/mantisbt-mcp-server), oder einfach `/mantisbt:setup` ausführen — der Skill prüft, ob die semantische Suche aktiv ist, und führt bei Bedarf durch die Aktivierung.

## Verwendung

```
/mantisbt:research wie ist der Status von Ticket 1234?
/mantisbt:create lege einen Bug an: Login schlägt nach Passwort-Reset fehl
/mantisbt:sync --info
/mantisbt:setup
```

Du kannst dein Anliegen auch einfach in natürlicher Sprache formulieren ("zeig offene P1-Bugs in Projekt X", "lege einen Feature-Wunsch für Dark Mode an") — der passende Skill wird anhand des Kontexts automatisch ausgelöst.

## Versionierung

Die Plugin-Version (`.claude-plugin/plugin.json`) ist unabhängig von der in `.mcp.json` gepinnten Version des MantisBT-MCP-Servers. Versionierungsregeln und der Release-Prozess sind in [DEVELOPMENT.md](DEVELOPMENT.md) beschrieben (auf Englisch).

## Datenschutz

Dieses Plugin hat kein eigenes Backend — siehe [PRIVACY.md](PRIVACY.md) (Englisch) für Details, welche Daten es verarbeitet und wie.

## Lizenz

[MIT](LICENSE) © Dominik Pesch
