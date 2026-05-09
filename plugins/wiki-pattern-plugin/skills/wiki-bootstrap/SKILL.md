---
name: wiki-bootstrap
description: "Crea una nuova wiki tecnica LLM-mantenuta da zero al pattern Karpathy esteso. Genera struttura cartelle (raw/inbox/wiki) e CLAUDE.md/index.md/log.md/tags.md adattati al dominio. Le skill operative (wiki-ingest/wiki-query/wiki-lint) provengono dal plugin globale, non vengono duplicate nella wiki. Parametri opzionali: dominio, path, lingua, tipo tassonomia, feature flags. Triggers: '/wiki-bootstrap', 'crea wiki', 'nuova wiki', 'crea una wiki per', 'bootstrap wiki', 'inizializza wiki'."
---

# Wiki Bootstrap

Crea una nuova wiki tecnica LLM-mantenuta al pattern, da zero, partendo dai template auto-contenuti del plugin in `${CLAUDE_PLUGIN_ROOT}/templates/`.

> **Importante**: questa skill **non genera** i file `.claude/skills/wiki-*/SKILL.md` nella nuova wiki. Le skill operative (`wiki-ingest`, `wiki-query`, `wiki-lint`) sono fornite dal plugin globale e funzionano in qualsiasi wiki che segue il pattern (rilevato dalla presenza di un `CLAUDE.md` conforme).

## Step 1 — Raccolta parametri

Se non forniti come argomenti, chiedi interattivamente:

1. **Dominio** — es. "Vue 4", "PostgreSQL 17", "AWS Lambda", "FastAPI"
2. **Path di destinazione** — percorso assoluto dove creare la wiki. Se la dir esiste e non è vuota, avvisa e chiedi conferma.
3. **Lingua testo** — default `it`. Altre opzioni: `en`, `es`, ecc.
4. **Tipo tassonomia** (`--type`):
   - `tech` (default) — cartelle: `<domain-dir>/`, `packages/`, `concepts/`, `integrations/`, `comparisons/`; tipi pagina: `feature`, `package`, `concept`, `integration`, `comparison`, `source`
   - `generic` — cartelle: `concepts/`, `entities/`, `references/`; tipi pagina: `concept`, `entity`, `reference`, `source`
5. **Feature flags** (default tutti attivi):
   - `--archive-pages [yes|no]` — abilita archive pages da query (default: yes)
   - `--provenance [light|strict|off]` — tracking provenance (default: light)
   - `--cross-linker [auto|manual]` — cross-linker post-ingest (default: auto)

**Shortcut**: se l'utente scrive `/wiki-bootstrap --domain "Vue 4" --path ~/wiki-vue4` passa direttamente allo Step 2 con i default per tutto il resto.

---

## Step 2 — Prepara variabili di rendering

| Placeholder | Valore | Esempio |
|---|---|---|
| `{{DOMAIN}}` | Nome dominio esatto | "Vue 4" |
| `{{DOMAIN_DIR}}` | Nome cartella wiki principale (kebab-case del dominio, senza versione) | "vue" |
| `{{LANG}}` | Codice lingua | "it" |
| `{{TODAY}}` | Data odierna YYYY-MM-DD | "2026-05-08" |
| `{{TYPE}}` | "tech" o "generic" | "tech" |
| `{{ARCHIVE_PAGES}}` | "yes" o "no" | "yes" |
| `{{PROVENANCE}}` | "light", "strict", o "off" | "light" |
| `{{CROSS_LINKER}}` | "auto" o "manual" | "auto" |

---

## Step 3 — Crea struttura cartelle

```bash
mkdir -p <WIKI_ROOT>/raw/assets
mkdir -p <WIKI_ROOT>/inbox
mkdir -p <WIKI_ROOT>/wiki/_meta

touch <WIKI_ROOT>/raw/assets/.gitkeep
touch <WIKI_ROOT>/inbox/.gitkeep

# Per type=tech:
mkdir -p <WIKI_ROOT>/wiki/{{DOMAIN_DIR}}
mkdir -p <WIKI_ROOT>/wiki/packages
mkdir -p <WIKI_ROOT>/wiki/concepts
mkdir -p <WIKI_ROOT>/wiki/integrations
mkdir -p <WIKI_ROOT>/wiki/comparisons

# Per type=generic:
mkdir -p <WIKI_ROOT>/wiki/concepts
mkdir -p <WIKI_ROOT>/wiki/entities
mkdir -p <WIKI_ROOT>/wiki/references
```

> **Nota**: NON creare `.claude/skills/`. Le skill operative arrivano dal plugin globale.

---

## Step 4 — Render del CLAUDE.md

1. **Read** `${CLAUDE_PLUGIN_ROOT}/templates/CLAUDE.md.tmpl`
2. **Sostituisci tutti i placeholder** `{{X}}` con i valori dello Step 2
3. **Rimuovi i blocchi conditional non attivi** (vedi sotto)
4. **Write** a `<WIKI_ROOT>/CLAUDE.md`

### Conditional blocks: regola di rendering

I template usano marker HTML-comment per delimitare blocchi opzionali:

```markdown
<!-- IF type=tech -->
contenuto solo per type=tech
<!-- END type=tech -->
```

Per ogni `<!-- IF <condition> -->...<!-- END <condition> -->`:
- Se la condizione è **vera** → mantieni il contenuto, rimuovi le linee marker
- Se la condizione è **falsa** → rimuovi tutto (marker + contenuto)

**Tabella di valutazione delle condizioni:**

| Marker | Vero quando |
|---|---|
| `IF type=tech` | parametro `--type` è "tech" |
| `IF type=generic` | parametro `--type` è "generic" |
| `IF archive-pages` | parametro `--archive-pages` è "yes" |
| `IF provenance` | parametro `--provenance` è "light" o "strict" (non "off") |
| `IF provenance-strict` | parametro `--provenance` è "strict" |
| `IF cross-linker-auto` | parametro `--cross-linker` è "auto" |
| `IF cross-linker-manual` | parametro `--cross-linker` è "manual" |

---

## Step 5 — Render index.md

1. **Read** `${CLAUDE_PLUGIN_ROOT}/templates/index.md.tmpl`
2. Sostituisci + risolvi conditional
3. **Write** a `<WIKI_ROOT>/index.md`

---

## Step 6 — Render log.md

1. **Read** `${CLAUDE_PLUGIN_ROOT}/templates/log.md.tmpl`
2. Sostituisci + risolvi conditional
3. **Write** a `<WIKI_ROOT>/log.md`

---

## Step 7 — Render wiki/_meta/tags.md

1. **Read** `${CLAUDE_PLUGIN_ROOT}/templates/tags.md.tmpl`
2. Sostituisci + risolvi conditional
3. **Write** a `<WIKI_ROOT>/wiki/_meta/tags.md`

---

## Step 8 — Output finale

```
✅ Wiki "{{DOMAIN}}" creata in <WIKI_ROOT>

📁 Struttura:
   raw/         → fonti immutabili (vuota, popola con le tue docs)
   inbox/       → appunti effimeri
   wiki/        → pagine compilate (vuoto, popolato dall'ingest)

⚙️ Skill operative disponibili (dal plugin globale):
   /wiki-ingest  → compila fonti in pagine wiki
   /wiki-query   → risponde dalla wiki
   /wiki-lint    → controllo qualità

🔧 Configurazione:
   type: <type> | lang: {{LANG}}
   archive-pages: <val> | provenance: <val> | cross-linker: <val>

📌 Prossimo passo:
   1. cd <WIKI_ROOT>
   2. Copia le tue fonti in raw/<topic>/
   3. Lancia /wiki-ingest per popolare la wiki
   4. Oppure: /wiki-ingest <url> per ingerire direttamente da web
```

---

## Edge case

- **Path esiste ed è non-vuota**: chiedi `"Directory non vuota. Continuare aggiungendo solo i file mancanti? [Y/n]"`. Se sì → non sovrascrivere file esistenti; segnala cosa è stato saltato.
- **DOMAIN ambiguo per la tassonomia**: scegli `tech`, deriva DOMAIN_DIR dal dominio.
- **Path relativo fornito**: converti in assoluto prima di procedere.
- **Lang non documentata**: usa la lingua indicata per il testo discorsivo ma tieni i nomi di comandi/file in inglese.
- **Template non trovato in `${CLAUDE_PLUGIN_ROOT}/templates/`**: errore fatale — il plugin non è installato correttamente.
- **Placeholder residuo `{{X}}` nel file scritto**: errore di rendering. Logga warning, segnala all'utente di verificare.
