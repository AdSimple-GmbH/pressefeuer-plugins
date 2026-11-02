# Pressefeuer für Claude Code

Mit diesem Plugin entwerfen und überarbeiten Agenturen ihre Pressemitteilungen für
[pressefeuer.at](https://pressefeuer.at) direkt in Claude Code, schicken Entwürfe zur
Kundenfreigabe und rufen Reichweitenberichte ab. Das Plugin verbindet Claude Code mit dem
offiziellen Pressefeuer-MCP-Server und bringt Arbeitsanleitungen (Skills) für den Redaktionsablauf mit.

## Voraussetzungen

- Ein Pressefeuer-Konto mit API-Zugang (derzeit Agency-Tarif).
- Claude Code ab Version 2.1.295.

## Installation

**Aus dem Pressefeuer-Marketplace** (in der Shell):

```bash
claude plugin marketplace add AdSimple-GmbH/pressefeuer-plugins
claude plugin install pressefeuer@pressefeuer
```

**Aus dem Anthropic-Verzeichnis:** sobald gelistet, auf claude.ai im Verzeichnis hinzufügen. Laut
[Claude-Code-Dokumentation](https://code.claude.com/docs/en/plugins/publish.md#how-a-listed-plugin-reaches-claude-code-users)
lädt Claude Code ein so hinzugefügtes Plugin als `pressefeuer@synced`.

## Anmeldung

Beim Aktivieren fragt Claude Code nach dem **Pressefeuer-Agentur-Schlüssel** (Dashboard → Admin →
Agent). Der Schlüssel wird im System-Schlüsselbund gespeichert und ausschließlich im HTTP-Header
`X-API-Key` an `https://pressefeuer.at/api/mcp` gesendet – nie in einer URL, nie in einer Datei des
Plugins. Ändern lässt er sich jederzeit unter `/plugin` → pressefeuer → Configure. Ohne Schlüssel
sendet Claude Code den Header leer; dann sind nur `initialize` und `tools/list` möglich.

Eine Anmeldung per Browser (OAuth, ohne Schlüssel) ist in Vorbereitung und heute noch nicht
verfügbar. Sobald pressefeuer.at sie anbietet, wird sie hier beschrieben.

## Skills

| Skill                           | Zweck                                                                                                          |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| `/pressefeuer:pressemitteilung` | Mandant und Tonalität laden, Pressemitteilung schreiben, als Entwurf speichern, auf Wunsch zur Freigabe senden |
| `/pressefeuer:berichte`         | Performance veröffentlichter Pressemitteilungen abfragen, unveränderliche Berichte erstellen und abrufen       |
| `/pressefeuer:verbindung`       | Verbindung prüfen und Anmeldung einrichten                                                                     |

Claude wählt die Skills auch selbst, wenn eine Anfrage passt.

## Werkzeuge des MCP-Servers

Lesend: `list_clients`, `get_client_persona`, `list_drafts`, `check_approval_status`,
`get_agency_credits`, `get_article_performance`, `get_publication_report`,
`list_publication_reports`.

Schreibend: `create_draft`, `update_draft`, `create_publication_report`, `generate_seo_metadata`,
`send_approval_request`, `publish_release`.

- `send_approval_request` verschickt sofort eine E-Mail mit Freigabelink an die angegebene Adresse
  und setzt den Entwurf auf „wartet auf Freigabe".
- `generate_seo_metadata` überschreibt SEO-Titel und -Beschreibung, auch bei bereits
  veröffentlichten Artikeln.
- `publish_release` gilt nur für freigegebene OTS-Importe des Betreibers und veröffentlicht sofort.
  Kunden-Pressemitteilungen werden im Pressefeuer-Dashboard veröffentlicht.

Alle Aufrufe sind auf die eigene Agentur beschränkt. Tarif-Freischaltung und Rate-Limit (derzeit
500 Anfragen pro Stunde und Agentur) setzt der Server unverändert durch.

## Was das Plugin ausführt und sendet

- **Netzwerk:** ausschließlich `https://pressefeuer.at/api/mcp` (MCP über HTTP). Claude Code ruft
  beim Verbinden `initialize` und `tools/list` auf. Gesendet werden die Werkzeugaufrufe mit ihren
  Parametern (z. B. Texte von Pressemitteilungen, Empfänger-Adressen) und, falls konfiguriert, der
  Agentur-Schlüssel im Header `X-API-Key`. Keine weiteren Ziele, keine Telemetrie.
- **Daten im Kontext:** Werkzeugergebnisse – etwa Entwurfstexte, Kundennamen, E-Mail-Adressen und
  Kommentare aus Freigaben – gelangen in den Kontext von Claude. Die Skills behandeln sie als Daten,
  nicht als Anweisungen, und geben Freigabelinks nicht aus.
- **Hook:** ein `PreToolUse`-Hook gibt vor `send_approval_request`, `publish_release` und
  `generate_seo_metadata` per `echo` einen festen Text mit der Entscheidung „nachfragen" aus. Claude
  Code fragt dann nach, sofern es Berechtigungsabfragen anzeigt; schlägt der Hook fehl, wird der
  Aufruf blockiert (`onFailure: block`). Der Hook liest keine Eingaben und sendet nichts. Wie jeder
  Plugin-Hook erhält sein Prozess die Plugin-Einstellungen als Umgebungsvariablen (u. a.
  `CLAUDE_PLUGIN_OPTION_API_KEY`); der Befehl verwendet sie nicht.
- **Keine** Skripte, Pakete, Downloads oder lokalen Server.

## Datenschutz und Support

- Datenschutz: <https://pressefeuer.at/datenschutz>
- AGB: <https://pressefeuer.at/agb>
- Dokumentation: <https://pressefeuer.at/docs/agents/mcp>
- Sicherheit: <https://pressefeuer.at/docs/agents/security>
- Support: <https://pressefeuer.at/contact> · office@pressefeuer.at

## Lizenz

MIT – siehe [LICENSE](LICENSE).
