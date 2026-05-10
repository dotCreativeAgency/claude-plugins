---
name: wiki-upgrade
description: "Migra una wiki tecnica esistente al pattern Karpathy esteso. Rileva lo stato attuale, produce un gap report, applica le modifiche meccaniche in auto e le modifiche di contenuto con conferma interattiva. Le skill operative provengono dal plugin globale: questa skill aggiorna solo CLAUDE.md/index.md/log.md/tags.md e rimuove eventuali skill operative inline obsolete. Triggers: '/wiki-upgrade', 'migra wiki', 'aggiorna wiki al pattern', 'upgrade wiki', 'converti wiki al pattern'."
---

# Wiki Upgrade

Migra una wiki tecnica esistente al pattern (raw/inbox/wiki + cascade + conflict annotation + archive pages + cross-linker + lint split), usando i template auto-contenuti in `${CLAUDE_PLUGIN_ROOT}/templates/`.

> **Importante**: questa skill **non installa** né mantiene i file `.claude/skills/wiki-{ingest,query,lint}/SKILL.md` nella wiki. Le skill operative arrivano dal plugin globale. Se la wiki ha skill operative inline (vecchio modello), la skill propone di rimuoverle (sono ridondanti rispetto al plugin globale).

## Step 1 — Raccolta parametri

Se non forniti:
- **Path della wiki** — percorso assoluto. Se non fornito, chiedi.
- **`--auto-deterministic`** (flag opzionale) — skippa le conferme per le modifiche meccaniche.

Determina dal CLAUDE.md esistente (o chiedi se non rilevabile):
- `{{DOMAIN}}` — nome del dominio
- `{{DOMAIN_DIR}}` — cartella wiki principale (rilevata dalle sotto-cartelle)
- `{{LANG}}` — lingua del testo discorsivo
- `{{TYPE}}` — `tech` o `generic`
- `{{TODAY}}`, feature flags — proponi default e chiedi conferma

---

## Step 2 — DETECTION: analisi stato attuale

Scansiona la wiki e classifica ogni componente come `present`, `partial`, o `missing`.

### 2a — Struttura cartelle

| Componente | Check | Stato |
|---|---|---|
| `raw/` | esiste ed è immutabile | present/missing |
| `inbox/` | esiste come top-level dir | present/missing |
| `wiki/` | esiste con sotto-cartelle | present/partial/missing |
| `wiki/_meta/tags.md` | esiste con formato registro | present/partial/missing |

### 2b — Skill inline (`.claude/skills/wiki-*/`)

| Componente | Cosa fare |
|---|---|
| `.claude/skills/wiki-ingest/SKILL.md` esiste | **Da rimuovere** (ora dal plugin globale) — proporre cleanup |
| `.claude/skills/wiki-query/SKILL.md` esiste | Idem |
| `.claude/skills/wiki-lint/SKILL.md` esiste | Idem |

### 2c — CLAUDE.md

Verifica presenza di:
- Sezione "Tre top-level dir, regole nette" → se no, "stile base"
- Schema frontmatter con `raw_source` + `raw_source_hash`
- Sezione "Conflict annotation"
- Sezione "Archive pages"
- Sezione "Provenance markers"
- Sezione "Modulo stato" + dir `wiki/stato/` con `todo.md` e `bug-aperti.md`

### 2d — Pagine wiki esistenti

Per un campione, verifica frontmatter:
- Campo `aliases:` presente?
- Pagine `src-*.md` con `raw_source:` e `raw_source_hash:`?

### 2e — index.md e log.md

- `index.md`: ha formato "categorie + colonna Updated"?
- `log.md`: append-only?

---

## Step 3 — GAP REPORT

Mostra all'utente cosa manca. Classifica:
- 🟢 `ok`
- 🟡 `partial`
- 🔴 `missing`
- ♻️ `obsolete` (skill inline da rimuovere)

**Meccaniche** (auto-fix sicuro):
- Creare cartelle mancanti (`inbox/`, `wiki/_meta/`)
- Creare `wiki/_meta/tags.md` se mancante (render del template)
- Aggiungere sezioni mancanti a `CLAUDE.md` (solo append, mai riscrivere)
- Aggiungere `.gitignore` se mancante
- **Rimuovere** `.claude/skills/wiki-{ingest,query,lint}/` se presenti (ridondanti col plugin globale; backup automatico in `.claude/skills.bak/`)

**Contenuto** (richiedono revisione):
- Aggiungere `aliases: []` al frontmatter di pagine esistenti
- Aggiungere `raw_source:` + `raw_source_hash:` a pagine `src-*` esistenti
- Riformattare `index.md` al nuovo formato

Chiedi: `"Trovati N item meccanici e M di contenuto. Procedere con i meccanici? [Y/n]"`.
Se `--auto-deterministic`: skippa la domanda.

---

## Step 4 — APPLICA modifiche meccaniche

### M1 — Crea inbox/ se mancante
```bash
mkdir -p <WIKI_ROOT>/inbox
touch <WIKI_ROOT>/inbox/.gitkeep
```

### M2 — Crea wiki/_meta/ e tags.md se mancanti
```bash
mkdir -p <WIKI_ROOT>/wiki/_meta
```
**Read** `${CLAUDE_PLUGIN_ROOT}/templates/tags.md.tmpl`, applica placeholder + conditional, **Write** a `<WIKI_ROOT>/wiki/_meta/tags.md`.

### M3 — Aggiorna CLAUDE.md (solo append)
Per ogni sezione mancante, aggiungi in fondo a `CLAUDE.md` *prendendola* dal template `${CLAUDE_PLUGIN_ROOT}/templates/CLAUDE.md.tmpl` (sezione corrispondente, con placeholder applicati).

Sezioni da aggiungere se mancanti:
1. "Le tre top-level dir, regole nette"
2. "Provenance — markers `^[inferred]`" (se provenance attivo)
3. "Conflict annotation tra fonti"
4. "Archive pages" (se archive-pages attivo)
5. "Tag taxonomy"
6. Regole generali

Verifica che non esista già (anche con nome diverso).

### M4 — Crea .gitignore se mancante
```
.obsidian/
*.local.md
```

### M5a — Crea wiki/stato/ se richiesto

Se l'utente ha confermato di abilitare il modulo `stato_pages` (chiedi se non rilevato dal CLAUDE.md):

```bash
mkdir -p <WIKI_ROOT>/wiki/stato
```

Crea `<WIKI_ROOT>/wiki/stato/todo.md` e `<WIKI_ROOT>/wiki/stato/bug-aperti.md` con frontmatter (`next_id: 1`, `data_aggiornamento: {{TODAY}}`) e corpo placeholder ("_Nessun TODO/bug aperto._"). Vedi `wiki-bootstrap` per il template esatto.

Se i file già esistono: skip silenzioso.

In CLAUDE.md, appendi la sezione "Modulo stato" del template. **Verifica prima che non esista già** (anche con titolo diverso ma contenuto equivalente): se è già presente, skip silenzioso — non duplicare. Il template `${CLAUDE_PLUGIN_ROOT}/templates/CLAUDE.md.tmpl` contiene **tre** blocchi `<!-- IF stato-pages -->`: appendere SOLO il blocco con heading `## Modulo stato` (l'unico che è una sezione vera e propria). Gli altri due blocchi sono frammenti contestuali (riga di tree e riga di tabella Operazioni) che non vanno appesi a EOF — se vuoi integrarli devi inserirli dentro la struttura esistente del CLAUDE.md della wiki, altrimenti ometti (M5a non rifà tree/tabella, è "solo append").

### M5 — Cleanup skill inline obsolete

Per ogni `.claude/skills/wiki-{ingest,query,lint}/` presente:
1. Mostra: `"Trovata skill inline obsoleta: .claude/skills/wiki-X/. Le skill operative ora arrivano dal plugin globale. Rimuovere? [Y/n/keep]"` (con `--auto-deterministic`: backup + rimozione automatica)
2. Se Y o auto:
   ```bash
   mkdir -p <WIKI_ROOT>/.claude/skills.bak
   mv <WIKI_ROOT>/.claude/skills/wiki-X <WIKI_ROOT>/.claude/skills.bak/
   ```
3. Se non resta nulla in `.claude/skills/`, rimuovi anche quella dir vuota.

### M6 — Append a log.md
```markdown
## [{{TODAY}}] refactor | Upgrade al pattern (wiki-upgrade)

- Modifiche meccaniche applicate: <lista>
- Skill inline rimosse (ora dal plugin globale): <lista o "nessuna">
- Modifiche contenuto pendenti: <lista o "nessuna">
- Plugin sorgente: ${CLAUDE_PLUGIN_ROOT} (wiki-pattern-plugin)
```

Output per ogni item: `✓ M<N> — <descrizione>`

---

## Step 5 — MODIFICHE DI CONTENUTO (interattive)

### C1 — Aggiungi `aliases: []` al frontmatter

Mostra: `"Trovate N pagine senza aliases. Aggiungere a tutte? [Y/n/select]"`. Se Y: inserisci `aliases: []` nel frontmatter dopo `tags:`.

### C2 — Aggiungi `raw_source` + `raw_source_hash` a pagine `src-*`

Per ogni pagina `wiki/src-*.md` senza `raw_source:`:
1. Cerca un file in `raw/` con slug simile
2. Match univoco → proponi: `"src-foo.md: raw_source = raw/bar/foo.md (sha1: abc123). Corretto? [Y/n/manuale]"`
3. Match assente → chiedi path manuale o skip
4. Aggiorna frontmatter dopo conferma

### C3 — Riformattazione index.md

Se `index.md` non è nel formato "categorie + colonna Updated":
1. Mostra formato attuale + anteprima target
2. Chiedi: `"Riformattare? Backup automatico in index.md.bak. [Y/n]"`
3. Se Y: riscrivi mantenendo link e descrizioni

---

## Step 6 — Output finale

```
✅ Upgrade completato per: <WIKI_ROOT>

🟢 Già presenti (N): raw/, wiki/, CLAUDE.md parziale, ...
🔧 Meccanici applicati (M): inbox/ creata, wiki/_meta/tags.md creato, ...
♻️ Skill inline rimosse (K): .claude/skills/wiki-{ingest,query,lint}/ → .claude/skills.bak/
✏️ Contenuto aggiornato (J): aliases aggiunti a N pagine, raw_source risolto per M src-*
⏭️ Saltati (W): <lista>

⚙️ Skill ora disponibili (dal plugin globale):
   /wiki-ingest  /wiki-query  /wiki-lint  /wiki-briefing (se stato_pages attivo)

📌 Prossimi passi consigliati:
   1. /wiki-lint — verifica coerenza post-upgrade
   2. /wiki-ingest — (se hai fonti in raw/ non ancora ingested)
```

---

## Edge case

- **Wiki completamente vuota (solo struttura base)**: equivale a un bootstrap parziale. Trattala come `/wiki-bootstrap` con `--skip-existing`.
- **CLAUDE.md non rilevabile come wiki**: avvisa `"CLAUDE.md non sembra appartenere a una wiki. Continuare comunque? [Y/n]"`.
- **Skill inline è una versione recente del pattern e l'utente potrebbe averla customizzata**: prima di rimuovere, confronta con il template del plugin. Se diverge in modo significativo, chiedi: `"Skill inline diverge dal template del plugin. Mostrare diff? Rimuovere comunque? [diff/Y/n]"`.
- **Conflitto di formato index.md**: backup `index.md.bak` automatico.
- **Path non trovato**: errore.
- **Template non trovato in `${CLAUDE_PLUGIN_ROOT}/templates/`**: errore fatale.
