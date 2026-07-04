# MantisBT-Plugin für Claude Code

*[English version](README.md)*

Ein [Claude Code](https://claude.com/product/claude-code)-Plugin, das den [MantisBT-MCP-Server](https://github.com/dpesch/mantisbt-mcp-server) in Claude Code integriert – damit lassen sich MantisBT-Tickets direkt aus einer Claude-Code-Sitzung recherchieren, anlegen und verwalten.

## Inhalt

- **MCP-Server** — startet `@dpesch/mantisbt-mcp-server` per `npx` und verbindet sich mit der REST-API deiner MantisBT-Instanz
- **Skills**
  - `/mantisbt:research` — Recherche zu Issues, Bugs und Projekt-Historie (nur lesend)
  - `/mantisbt:create` — geführtes Anlegen eines neuen Tickets (Bug, Feature, Task)
  - `/mantisbt:sync` — Status des MantisBT-Metadaten-Caches prüfen oder aktualisieren
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

## Verwendung

```
/mantisbt:research wie ist der Status von Ticket 1234?
/mantisbt:create lege einen Bug an: Login schlägt nach Passwort-Reset fehl
/mantisbt:sync --info
```

Du kannst dein Anliegen auch einfach in natürlicher Sprache formulieren ("zeig offene P1-Bugs in Projekt X", "lege einen Feature-Wunsch für Dark Mode an") — der passende Skill wird anhand des Kontexts automatisch ausgelöst.

## Versionierung

Die Plugin-Version (`.claude-plugin/plugin.json`) ist unabhängig von der in `.mcp.json` gepinnten Version des MantisBT-MCP-Servers. Versionierungsregeln und der Release-Prozess sind in [DEVELOPMENT.md](DEVELOPMENT.md) beschrieben (auf Englisch).

## Lizenz

[MIT](LICENSE) © Dominik Pesch
