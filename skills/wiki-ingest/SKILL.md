---
name: wiki-ingest
description: "Ingerisci una fonte (file in raw/, file in inbox/, o URL) in una wiki LLM-mantenuta del pattern Karpathy esteso. Crea/aggiorna pagine src-* e pagine compilate; esegue cascade update, conflict annotation, provenance marking, cross-linker; aggiorna il registro tag. Senza argomenti scansiona inbox/ poi raw/ pendenti via stato derivato + hash. La skill legge il CLAUDE.md della wiki corrente per dominio e feature flags. Triggers: '/wiki-ingest', 'ingerisci', 'ingest', 'aggiungi alla wiki', 'processa raw/', 'processa inbox'."
---

# Wiki Ingest

Ingerisci fonti nella wiki LLM-mantenuta della directory di lavoro corrente.

**Pre-requisito**: la directory deve contenere `CLAUDE.md` con lo schema del pattern wiki (raw/inbox/wiki + tipi pagina + frontmatter standard). Se manca → suggerisci `/wiki-bootstrap` o `/wiki-upgrade`.

**Configurazione runtime**: leggi `CLAUDE.md` della cwd per:
- Dominio della wiki (titolo della pagina, sezione "Scopo")
- Tassonomia (sotto-cartelle in `wiki/`, tipi di pagina)
- Feature flags (presenza di sezioni "Archive pages", "Provenance markers", "Tag taxonomy")
- Lingua (sezione "Lingua")

Se una feature non è documentata in CLAUDE.md → skip dello step relativo (es. niente provenance markers se la wiki non li usa).

## Modalità di invocazione

| Input | Cosa fai |
|---|---|
| `/wiki-ingest` (no args) | Modalità auto: processa `inbox/` poi pendenti `raw/` |
| `/wiki-ingest <path>` | Processa singolo file (raw/ o inbox/) |
| `/wiki-ingest <url>` | Scarica in `raw/<topic>/<slug>.md` poi processa |

---

## Modalità AUTO (no args)

### Step A1 — Scan inbox/
```bash
find inbox -maxdepth 2 -name "*.md" -type f 2>/dev/null
```
Se non vuoto: chiedi `"Trovati N appunti in inbox/. Processarli? [Y/n/peek/select]"`. `peek` mostra prima riga di ogni file.

### Step A2 — Scan raw/ pendenti
```bash
# Tutti i candidati raw/
find raw -name "*.md" -type f -not -path "*/assets/*"
# raw_source già processati (estratti dai frontmatter src-*)
grep -h "^raw_source:" wiki/src-*.md 2>/dev/null | sed 's/raw_source: *//'
```
Calcola differenza = pendenti per nuovo ingest.

Per ogni file `raw/` già ingerito: confronta hash corrente vs `raw_source_hash:`:
```bash
sha1sum raw/path/to/file.md | awk '{print $1}'
```
Se diverso → segna come "modificato, ri-ingest".

Mostra lista categorizzata: `Nuovi (N), Modificati (M)`. Chiedi conferma o `select`.

### Step A3 — Processa
- Inbox prima (vedi sezione "Processa file da inbox/")
- Poi raw/ pendenti (vedi sezione "Processa file da raw/")

Se entrambi vuoti: `"Nessun documento da processare. Aggiungi file in raw/ o inbox/."`

---

## Processa file da raw/

### Step 1 — Leggi e analizza fonte
- `Read` del file
- Identifica titolo, data pubblicazione (se presente), versione/topic principale
- Calcola `sha1sum` del file → `raw_source_hash`

### Step 2 — Crea/aggiorna pagina src-*
Path: `wiki/src-<slug>.md` (slug = kebab dal titolo, max 60 char).

Frontmatter:
```yaml
---
title: "<titolo fonte>"
type: source
sources: []
tags: [<tag rilevanti, max 5>]
updated: <oggi YYYY-MM-DD>
raw_source: raw/<path>
raw_source_hash: <sha1>
---
```

Corpo:
```markdown
## Fonte originale
<URL o riferimento se presente nel file>

## Data ingest
<oggi>

## Punti chiave estratti
- ...

## Pagine wiki create/aggiornate
- [[pagina-1]] — creata
- [[pagina-2]] — aggiornata (sezione X)

## Contraddizioni o dati nuovi rispetto al corpus esistente
- ...
```

Se `wiki/src-<slug>.md` già esiste con stesso `raw_source_hash`: skip (no-op, log warning).
Se esiste con hash diverso: ri-ingest → aggiorna corpo, ricalcola cascade.

### Step 3 — Identifica pagine wiki impattate
Per ogni concetto/feature trovato nella fonte:
- Esiste già una pagina wiki? Lista pagine candidate (`grep -l` per termini chiave in `wiki/`)
- Se sì → da aggiornare
- Se no → da creare (scegli sotto-cartella in base alla tassonomia letta da CLAUDE.md)

### Step 4 — Crea/aggiorna pagine compilate
Per ogni pagina interessata:

**Frontmatter pagine normali:**
```yaml
---
title: "..."
type: <uno dei tipi del progetto, vedi CLAUDE.md>
tags: [...]
aliases: [...]
sources: [src-<slug>, ...]
updated: <oggi>
---
```

**Corpo (template tipici per tipo):**

Per `feature` / `concept` / `integration` / `entity`:
```markdown
## Panoramica
## Come funziona
## Esempi pratici
## Configurazione (opzionale)
## Gotcha e note importanti
## Vedi anche
- [[pagina-correlata]] — relazione
```

Per `package`:
```markdown
## Panoramica
## Installazione
## Configurazione
## Utilizzo base
## Esempi avanzati
## Compatibilità
## Alternative
## Vedi anche
```

Per `comparison`:
```markdown
## Soggetti confrontati
## Tabella riassuntiva
## Analisi dettagliata
## Quando scegliere X
## Quando scegliere Y
## Verdetto
## Vedi anche
```

**REGOLE durante la scrittura:**

- **Provenance** *(solo se la wiki ha sezione "Provenance" in CLAUDE.md)*: per ogni claim non letteralmente nella fonte → marca `^[inferred: motivo]`. Se fonti disagree → `^[ambiguous: dettaglio]`. Default = no marker.
- **Sintesi**: estrai essenziale + esempi concreti — NON copiare la docs.
- **Cross-reference**: usa `[[wikilink]]` ovunque ci sia una pagina correlata. Non duplicare contenuto: se un concetto ha pagina propria, linka, non rispiegare.
- **Codice**: verifica sintassi/API per la versione corrente del dominio.
- **Lingua**: vedi sezione "Lingua" del CLAUDE.md della wiki corrente. Inglese per codice/comandi/identificatori.

### Step 5 — CASCADE SCAN (obbligatorio)

Dopo aver scritto la pagina primaria:

1. Scansiona altre pagine nella stessa sotto-cartella (`wiki/<topic>/`):
   ```bash
   ls wiki/<topic>/*.md
   ```
   Per ognuna: leggi e valuta se la nuova fonte ne modifica fatti/esempi. Se sì → aggiorna sezione interessata, aggiorna `updated:` nel frontmatter.

2. Scansiona `index.md` per pagine in **altre** cartelle che potrebbero essere correlate per topic. Per ognuna: stessa valutazione.

3. **Pagine `archive: true` sono ESCLUSE dal cascade** (sono snapshot point-in-time; salta se la wiki non supporta archive).

### Step 6 — CONFLICT ANNOTATION

Se la nuova fonte contraddice contenuto esistente in qualsiasi pagina toccata, **non sostituire silenziosamente**. Aggiungi blocco standardizzato:

```markdown
> ⚠️ **Conflitto fonti**
> - [[src-<vecchia>]] (<data vecchia>): "<claim vecchio>"
> - [[src-<nuova>]] (<data nuova>): "<claim nuovo>"
>
> Versione corrente: **<scelta>**.
```

Se il conflitto vive su due pagine diverse: aggiungi annotazione in entrambe + cross-link reciproco.

### Step 7 — REGISTRO TAG

Per ogni tag usato nelle pagine create/aggiornate:
1. Leggi `wiki/_meta/tags.md`
2. Se il tag NON esiste → append una riga: `- \`<tag>\` — <descrizione una frase>. Usato in <N> pagine.`
3. Se esiste → aggiorna conteggio (opzionale, può farlo lint)

Verifica limite ≤5 tag per pagina (warning se superato, non blocca).

### Step 8 — CROSS-LINKER

*(skip se la wiki non supporta cross-linker auto — vedi CLAUDE.md sezione Operazioni / Regole generali)*

Per ogni pagina creata/aggiornata:
1. Estrai `title` + `aliases` (lista termini)
2. Per ogni termine, scansiona TUTTE le altre pagine wiki:
   ```bash
   grep -rn -F "<termine>" wiki/ --include="*.md"
   ```
3. Per ogni occorrenza valida (case-sensitive, word boundary, NON dentro code block ` ``` `, NON dentro `` ` `` inline, NON in frontmatter, NON già linkata `[[...]]`):
   - Se match al titolo è **univoco** nel vault → linka SOLO la prima occorrenza nella pagina con `[[<slug>]]` (o `[[<slug>|<termine>]]` se diverso dal titolo)
   - Se ambiguo → segnala in output, NO fix
4. Output:
   ```
   🔗 Cross-link scan
     ✓ wiki/<topic>/x.md: linkato "<Term>" → [[term-slug]]
     ⚠️ wiki/<topic>/y.md: "<Term>" ambiguo, skip
   ```

### Step 9 — Aggiorna index.md

Per ogni pagina creata o materialmente aggiornata:
- Trova la sezione di categoria corretta
- Aggiungi/aggiorna riga: `- [[<slug>]] (<updated>) — <descrizione una riga>`
- Per pagine archive: prefisso `[Archived]` nella descrizione
- Aggiorna conteggio totale pagine in cima a index.md

### Step 10 — Append log.md

```markdown
## [YYYY-MM-DD] ingest | <titolo fonte primaria>
- src: [[src-<slug>]]
- Created: [[pagina-nuova-1]], [[pagina-nuova-2]]
- Updated: [[pagina-cascade-1]] (sezione X)
- Tags new: <tag1>, <tag2>
- Conflicts: <sintesi se presenti, altrimenti omesso>
```

---

## Processa file da inbox/

Stesso flusso da Step 1 a Step 10, con queste differenze:

- `raw_source:` è **omesso** nella pagina src-* (preferisci: niente src-* per gli inbox).
- **Decidi se promuovere a raw/**: se il contenuto è autorevole/persistente, proponi `"Questo sembra contenuto autorevole. Promuoverlo a raw/<topic>/<slug>.md? [Y/n]"`. Se sì: `mv inbox/X.md raw/<topic>/X.md` e procedi come ingest da raw/.
- Se NON promosso: dopo aver compilato le pagine wiki, **cancella** il file da inbox/. Append a log con operazione `inbox-process` invece di `ingest`.

---

## Modalità file singolo

`/wiki-ingest <path>`: salta scan iniziale, processa direttamente quel file con il flusso appropriato (raw/ o inbox/ a seconda di dove sta).

---

## Modalità URL

`/wiki-ingest <url>`:
1. Scarica con `WebFetch` (o suggerisci all'utente di fare paste se autenticato)
2. Determina topic e slug (chiedi se ambiguo)
3. Salva in `raw/<topic>/YYYY-MM-DD-<slug>.md` con header metadati (URL, data fetch)
4. Procedi con flusso raw/

---

## Output finale dell'ingest

```
✅ Ingest completato

📥 Fonti processate: <N>
📄 Pagine create: <list>
✏️ Pagine aggiornate: <list>
🔗 Cross-link aggiunti: <N>
⚠️ Conflitti annotati: <N>
🏷️ Tag nuovi: <list>

📓 Log: aggiornato
📑 Index: aggiornato
```

---

## Errori e edge case

- **CLAUDE.md mancante o non-wiki**: avvisa `"Questa directory non sembra una wiki LLM-mantenuta. Lancia /wiki-bootstrap per crearne una nuova o /wiki-upgrade per migrare una wiki esistente."`
- **File raw/ esiste ma src-* corrispondente è marcato già hash uguale**: skip silenzioso, log debug
- **inbox/ contiene non-markdown**: skip e segnala
- **URL fallisce fetch**: chiedi paste manuale
- **Topic ambiguo**: chiedi all'utente in quale sotto-cartella collocare
- **Conflitto irrisolvibile**: annota e procedi, NON bloccare
