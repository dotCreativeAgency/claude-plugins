# wiki-pattern-plugin

Plugin Claude Code per **costruire e mantenere wiki tecniche LLM-mantenute**, basato sul pattern LLM Wiki di Karpathy con 11 estensioni.

5 skill in totale — 2 meta (bootstrap/upgrade) + 3 operative (ingest/query/lint) — installate **una sola volta** globalmente, funzionanti in qualsiasi wiki che segue il pattern.

## Cos'è il pattern

Ispirato a [karpathy/llm-wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) e [obsidian-wiki](https://github.com/Ar9av/obsidian-wiki), con scelte custom ottimizzate per wiki tecniche monodomain:

- **Tre dir top-level con regole nette**: `raw/` immutabile, `inbox/` effimera, `wiki/` LLM-owned
- **Tracking via stato derivato**: `raw_source` + `raw_source_hash` (sha1) nei frontmatter `src-*`, nessun manifest esterno
- **Ingest con cascade**: dopo ogni fonte, aggiorna automaticamente le pagine correlate
- **Conflict annotation esplicita**: blocco standard `> ⚠️ Conflitto fonti` con attribuzione
- **Provenance markers**: `^[inferred]` per claim non da fonte, `^[ambiguous]` per fonti in disaccordo
- **Archive pages**: sintesi da query salvate come snapshot immune a cascade (`archive: true`)
- **Cross-linker post-ingest**: wikilinks auto-aggiunti su match univoci
- **Lint split**: deterministico (auto-fix) + euristico (report)
- **Registro tag leggero**: `wiki/_meta/tags.md` auto-aggiornante

## Le 5 skill

| Skill | Scopo | Quando si attiva |
|---|---|---|
| `/wiki-bootstrap` | Crea una nuova wiki da zero | All'inizio: prima volta che vuoi una wiki nuova |
| `/wiki-upgrade` | Migra una wiki esistente al pattern | Quando hai una wiki "stile base" da modernizzare |
| `/wiki-ingest` | Compila una fonte raw/ o inbox/ in pagine wiki | Dopo aver droppato fonti |
| `/wiki-query` | Risponde a una domanda dalla wiki, opzionale archive | Per consultare la conoscenza accumulata |
| `/wiki-lint` | Health-check con auto-fix + report euristico | Periodicamente, dopo ingest in batch |

Le 3 skill operative leggono il `CLAUDE.md` della wiki corrente per capire dominio, tassonomia e feature flags. Funzionano su qualsiasi wiki conforme al pattern senza necessità di duplicare i file SKILL nelle singole wiki.

## Installazione

### Via Claude Code marketplace (raccomandato)

Da dentro Claude Code:

```
/plugin marketplace add dotCreativeAgency/wiki-pattern-plugin
/plugin install wiki-pattern-plugin@wiki-pattern
```

Riapri Claude Code se non vedi i 5 comandi (`/wiki-bootstrap`, `/wiki-upgrade`, `/wiki-ingest`, `/wiki-query`, `/wiki-lint`).

### Manuale (clone + symlink)

```bash
git clone https://github.com/dotCreativeAgency/wiki-pattern-plugin
cd wiki-pattern-plugin

# Installa tutte le 5 skill globalmente
for s in wiki-bootstrap wiki-upgrade wiki-ingest wiki-query wiki-lint; do
  ln -sfn "$(pwd)/skills/$s" ~/.claude/skills/$s
done
```

(Symlink permettono di ricevere automaticamente gli aggiornamenti via `git pull`.)

Oppure registra il plugin localmente come marketplace:

```
/plugin marketplace add /path/to/wiki-pattern-plugin
/plugin install wiki-pattern-plugin@wiki-pattern
```

## Uso

### Creare una nuova wiki da zero

```
/wiki-bootstrap
```

Oppure con parametri:

```
/wiki-bootstrap --domain "Vue 4" --lang it --path ~/wiki-vue4
```

### Migrare una wiki esistente

```
/wiki-upgrade --path ~/mia-wiki-esistente
```

### Operazioni su una wiki

`cd` nella directory della wiki, poi:

```
/wiki-ingest                              # processa inbox/ + raw/ pendenti
/wiki-ingest raw/topic/source.md          # singolo file
/wiki-ingest https://example.com/docs     # URL

/wiki-query Cosa so su X?
/wiki-query Confronta A e B               # poi: "salva questa risposta" → archive page

/wiki-lint                                # auto-fix deterministico + report euristico
```

## Parametri configurabili (per wiki-bootstrap)

| Parametro | Default | Opzioni | Descrizione |
|---|---|---|---|
| `--domain` | (chiesto) | qualsiasi | Nome del dominio tecnico (es. "Laravel 13", "PostgreSQL 17") |
| `--path` | (chiesto) | percorso | Dove creare la wiki |
| `--lang` | `it` | `it`, `en`, ... | Lingua del testo discorsivo |
| `--type` | `tech` | `tech`, `generic` | Tassonomia pagine |
| `--archive-pages` | `yes` | `yes`, `no` | Abilita archive pages da query |
| `--provenance` | `light` | `light`, `strict`, `off` | Tracking provenance |
| `--cross-linker` | `auto` | `auto`, `manual` | auto = step finale ingest, manual = solo via lint |

## Struttura generata in una nuova wiki

```
<wiki-root>/
├── CLAUDE.md                     ← schema + convenzioni (adattato al dominio)
├── index.md                      ← catalogo pagine (categorie + Updated)
├── log.md                        ← log append-only
├── raw/                          ← fonti immutabili
│   └── assets/
├── inbox/                        ← appunti effimeri
└── wiki/
    ├── _meta/
    │   └── tags.md               ← registro tag
    ├── <domain-dir>/             ← feature/sottosistemi
    ├── concepts/
    ├── integrations/
    ├── comparisons/
    └── src-*.md                  ← pagine source
```

> **Nota**: `wiki-bootstrap` NON crea `.claude/skills/` nella wiki. Le 5 skill arrivano dal plugin globale.

## Struttura del plugin (questo repo)

```
wiki-pattern-plugin/
├── .claude-plugin/
│   ├── plugin.json              ← manifest (5 skill)
│   └── marketplace.json         ← manifest marketplace
├── skills/
│   ├── wiki-bootstrap/SKILL.md  ← meta: crea nuova wiki
│   ├── wiki-upgrade/SKILL.md    ← meta: migra wiki esistente
│   ├── wiki-ingest/SKILL.md     ← operativa: compila fonti
│   ├── wiki-query/SKILL.md      ← operativa: risponde dalla wiki
│   └── wiki-lint/SKILL.md       ← operativa: health-check
├── templates/                   ← rendered da bootstrap/upgrade
│   ├── CLAUDE.md.tmpl
│   ├── index.md.tmpl
│   ├── log.md.tmpl
│   └── tags.md.tmpl
├── CLAUDE.md
├── README.md
└── LICENSE
```

## Compatibilità con la versione 0.1.x

Wiki create con la versione 0.1.x del plugin hanno `.claude/skills/wiki-{ingest,query,lint}/SKILL.md` inline. Quelle skill inline **continuano a funzionare** ma sono **ridondanti** rispetto al plugin globale 0.2+. Per fare cleanup:

```
/wiki-upgrade --path /path/to/old-wiki
```

L'upgrade rileva le skill inline obsolete e propone di rimuoverle (backup automatico in `.claude/skills.bak/`).

## Crediti

Pattern basato su:
- Andrej Karpathy — [LLM Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [astro-han/karpathy-llm-wiki](https://github.com/astro-han/karpathy-llm-wiki)
- [ar9av/obsidian-wiki](https://github.com/Ar9av/obsidian-wiki)

Convention plugin Claude Code mutuate da [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills).
