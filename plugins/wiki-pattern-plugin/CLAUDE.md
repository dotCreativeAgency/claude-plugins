# wiki-pattern-plugin

Plugin Claude Code per creare e migrare **wiki tecniche LLM-mantenute** al pattern ibrido.

## Pattern di riferimento

Il pattern è documentato nelle skill di questo plugin. In sintesi:

**Tre directory top-level:**
- `raw/` — fonti autorevoli, **immutabile**
- `inbox/` — appunti effimeri, processati e rimossi dopo ingest
- `wiki/` — output compilato, **LLM-owned**

**Tre operazioni (slash command):**
- `/wiki-ingest` — compila fonti in pagine wiki (cascade + conflict annotation + cross-linker)
- `/wiki-query` — risponde dalla wiki; su richiesta salva come archive page
- `/wiki-lint` — auto-fix deterministico + report euristico

**Tracking via stato derivato:** ogni pagina `wiki/src-*.md` ha `raw_source:` + `raw_source_hash:` (sha1). Nessun manifest esterno.

## Skill disponibili

| Comando | Cosa fa |
|---|---|
| `/wiki-bootstrap` | Crea una nuova wiki tecnica da zero |
| `/wiki-upgrade` | Migra una wiki esistente al pattern |

## Installazione

Da Claude Code:

```
/plugin marketplace add dotCreativeAgency/claude-plugins
/plugin install wiki-pattern-plugin@dotcreativeagency-plugins
```

Oppure registra il marketplace localmente puntando a questo repo:

```
/plugin marketplace add /path/to/claude-plugins
/plugin install wiki-pattern-plugin@dotcreativeagency-plugins
```

## Lingua e convenzioni

- Testo discorsivo dell'output: **italiano** (default, override con `--lang en`)
- Codice, identificatori, comandi: **inglese**
