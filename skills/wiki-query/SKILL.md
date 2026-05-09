---
name: wiki-query
description: "Rispondi a una domanda attingendo a una wiki LLM-mantenuta del pattern Karpathy esteso (preferendo i contenuti wiki rispetto al training). Cita pagine con [[wikilink]]. Su richiesta esplicita ('salva', 'archivia') salva la risposta come archive page (flag archive true, immune a cascade) — solo se la wiki supporta archive pages. Triggers: '/wiki-query', 'cosa sai su', 'what do I know about', 'cerca nella wiki', 'secondo la wiki'."
---

# Wiki Query

Rispondi a domande tecniche attingendo prioritariamente alla wiki LLM-mantenuta della directory di lavoro corrente.

**Pre-requisito**: la directory deve contenere `CLAUDE.md` con lo schema del pattern wiki. Se manca → suggerisci `/wiki-bootstrap` o segnala che la cwd non è una wiki.

**Configurazione runtime**: leggi `CLAUDE.md` per dominio, tassonomia, feature flags. Il sotto-flusso archive viene attivato solo se la wiki ha la sezione "Archive pages" nel CLAUDE.md.

## Flusso base

### Step 1 — Locate
```bash
cat index.md
```
Identifica le pagine candidate per topic della domanda.

### Step 2 — Read
Leggi le pagine pertinenti (`Read` su ognuna). Se servono più pagine → leggile in parallelo.

Cerca anche claim correlati cross-topic:
```bash
grep -rln -i "<termine chiave>" wiki/ --include="*.md"
```

### Step 3 — Synthesize
Componi la risposta:
- **Prefer wiki over training**: se la wiki copre il punto, cita la wiki, non il tuo training
- **Cita con `[[wikilink]]`**: ogni claim non banale → link alla pagina sorgente
- **Distingui provenance** *(se la wiki usa `^[inferred]`)*: se la pagina cita un claim `^[inferred]`, segnala nella risposta ("attenzione: la wiki dichiara questa parte come inferenza, non da fonte ufficiale")
- **Conflitti**: se trovi blocchi `> ⚠️ Conflitto fonti` rilevanti, riportali esplicitamente
- **Pagine archive** *(se la wiki le supporta)*: se citi una `[Archived]`, segnala data archive (potrebbe essere obsoleta)

### Step 4 — Output
Risposta nella conversazione. **Non scrivere file** salvo richiesta esplicita di archive.

### Step 5 — Gap report (se applicabile)
Se durante la query scopri che:
- Un concetto è citato spesso ma manca pagina propria
- La wiki copre solo parzialmente la domanda
- Una fonte rilevante non è ancora stata ingerita

→ Segnala in coda alla risposta: `"💡 Gap rilevato: [...]. Vuoi che esegua un ingest di [...]?"`

---

## Sotto-flusso ARCHIVE

*Disponibile solo se la wiki ha sezione "Archive pages" nel CLAUDE.md.*

Triggerato **solo** da richiesta esplicita dell'utente: "salva", "archivia", "save this", "save to wiki", "metti questa risposta nella wiki".

### Step A1 — Determina collocazione
Topic della risposta → cartella appropriata (es. confronto → `wiki/comparisons/`, sintesi feature → cartella feature appropriata, integrazione → `wiki/integrations/`). Vedi sotto-cartelle elencate in CLAUDE.md.

### Step A2 — Slug e file
`<slug>.md` derivato dall'argomento della query (kebab-case, descrittivo).
Path: `wiki/<topic>/<slug>.md`.

Se file esiste già → chiedi: "Esiste già `<path>`. Sovrascrivere o creare nuovo (`<slug>-2.md`)?"

### Step A3 — Frontmatter archive

```yaml
---
title: "<titolo descrittivo>"
type: <tipo semantico più adatto, vedi CLAUDE.md>
tags: [...]
aliases: []
sources: [<pagina-wiki-1>, <pagina-wiki-2>, ...]   # pagine wiki citate
updated: <oggi>
archive: true
archived_at: <oggi>
archived_from_query: "<domanda originale dell'utente>"
---
```

Niente campo `raw_source` (l'archive non viene da raw/).

### Step A4 — Corpo
Scrivi la risposta come pagina wiki ben strutturata. Mantieni i `[[wikilink]]` alle pagine citate. Aggiungi sezione finale:

```markdown
## Origine
Questa pagina è un'**archive** generata da query del <data>. Snapshot del ragionamento al momento.
La wiki sorgente è viva e potrebbe essere evoluta dopo questa data.

**Domanda originale**: "<query>"

## Vedi anche
- [[pagina-citata-1]] — <relazione>
- [[pagina-citata-2]] — <relazione>
```

### Step A5 — Aggiorna index.md
Trova sezione di categoria. Aggiungi riga con prefisso `[Archived]`:
```markdown
- [[<slug>]] (<oggi>) — [Archived] <descrizione una riga>
```

### Step A6 — Append log.md
```markdown
## [YYYY-MM-DD] archive | <titolo pagina>
- query: "<domanda originale>"
- sources: [[pagina-1]], [[pagina-2]]
```

### Step A7 — Conferma
```
📦 Archive page creata: wiki/<topic>/<slug>.md
✓ index.md aggiornato (prefisso [Archived])
✓ log.md aggiornato

ℹ️ Questa pagina è IMMUNE a cascade — futuri ingest non la modificheranno.
   Se vuoi rigenerarla con info aggiornate, ri-fai la query e archivia di nuovo.
```

---

## Edge case

- **CLAUDE.md mancante o non-wiki**: avvisa `"Questa directory non sembra una wiki. Per usare /wiki-query servire una wiki LLM-mantenuta. Lancia /wiki-bootstrap per crearne una."`
- **Wiki vuota / index.md vuoto**: rispondi dal training segnalando esplicitamente `"⚠️ Nessuna pagina nella wiki copre questo. Risposta dal training generale, non dalla wiki."` e suggerisci un ingest.
- **Solo pagine src-* trovate, niente compilato**: leggi anche src-* ma segnala `"💡 Esiste solo la fonte raw — il contenuto compilato non c'è ancora. Vuoi che lo compili con un ingest?"`.
- **Domanda fuori dominio**: rispondi normalmente segnalando che non è scopo della wiki.
- **Richiesta archive ma wiki non supporta archive pages**: avvisa `"Questa wiki non supporta archive pages (CLAUDE.md non documenta la feature). Posso solo rispondere senza salvare. Per abilitare archive, lancia /wiki-upgrade."`
