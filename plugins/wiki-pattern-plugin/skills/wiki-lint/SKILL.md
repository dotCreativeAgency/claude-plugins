---
name: wiki-lint
description: "Esegui controllo di salute di una wiki LLM-mantenuta del pattern Karpathy esteso, con due categorie distinte: deterministici (auto-fix immediato di index/link/raw_source/See Also) ed euristici (solo report su contraddizioni, claim datati, orfane, provenance speculativa, tag duplicati, raw/ non ingerito o modificato). Output strutturato con sezioni Auto-fixed, Da rivedere, Summary. Triggers: '/wiki-lint', 'lint wiki', 'controlla wiki', 'audit wiki', 'verifica wiki', 'health check wiki'."
---

# Wiki Lint

Controllo di salute di una wiki LLM-mantenuta della directory di lavoro corrente.

**Pre-requisito**: la directory deve contenere `CLAUDE.md` con lo schema del pattern wiki. Se manca → suggerisci `/wiki-bootstrap` o `/wiki-upgrade`.

**Configurazione runtime**: leggi `CLAUDE.md` per feature flags. Skip dei check non applicabili (es. H7 provenance se la wiki non usa `^[inferred]`; H11 archive se la wiki non supporta archive pages).

Due categorie di check con autorità diversa:
- **Deterministici** → auto-fix immediato senza chiedere
- **Euristici** → solo report (richiedono giudizio umano)

---

## CHECK DETERMINISTICI (auto-fix)

### D1 — Index ↔ filesystem

```bash
# Lista file wiki effettivi
find wiki -type f -name "*.md" -not -path "wiki/_meta/*" | sort
# Pagine citate in index.md (escludi inline code e frontmatter)
grep -oE "\[\[[a-z0-9_/-]+\]\]" index.md | sort -u
```

**Esclusioni per il match degli `[[wikilink]]` in index.md:**
- Link dentro inline code (` `[[esempio]]` `) — sono testo descrittivo del formato, non entry reali
- Link dentro blocchi codice (` ``` ... ``` `)
- Link dentro frontmatter YAML

Regole:
- File esiste ma manca da index.md → aggiungi entry. Per `Updated` usa il campo `updated:` del frontmatter; fallback a `stat -c %y` del file
- Entry in index.md punta a file inesistente → marca `[MISSING]` accanto al titolo (NON cancellare la riga, lascia decidere all'utente)

### D2 — Link interni rotti

Per ogni link `[[<slug>]]` o `[testo](<path>)` nel corpo delle pagine wiki/ (esclusi `wiki/_meta/`, `index.md`, `log.md`):

```bash
grep -roE "\[\[[a-z0-9-]+\]\]" wiki/ --include="*.md"
```

Per ogni target inesistente:
- Cerca file con stesso nome altrove nella wiki:
  ```bash
  find wiki -name "<slug>.md"
  ```
- Esattamente 1 match → fix automatico (riscrivi il link col nuovo path)
- 0 o ≥2 match → **report**, non fix

Escludi link in blocchi codice e frontmatter.

### D3 — Riferimenti `raw_source:` rotti

```bash
grep -h "^raw_source:" wiki/src-*.md | sed 's/raw_source: *//' | while read p; do
  [ -f "$p" ] || echo "MISSING: $p"
done
```

Per ogni `raw_source:` puntato a file inesistente:
- Cerca in raw/ con stesso basename → se univoco, fix
- 0 o ≥2 match → report

### D4 — See Also ovvi mancanti

Per pagine nella stessa sotto-cartella `wiki/<topic>/`:
- Se A linka B nel corpo via `[[B]]` ma B non ha A nella sezione `## Vedi anche` → aggiungi entry reciproca
- Solo per link "principali" (≥2 occorrenze nel corpo o nel titolo di sezione)

### D5 — Conteggi tag in registro

`wiki/_meta/tags.md`: per ogni tag, conta pagine effettive che lo usano:
```bash
grep -lE "^tags:.*\b<tag>\b" $(find wiki -name '*.md') | wc -l
```
Aggiorna conteggio in registro.

### D6 — Cross-link mancanti (se modalità manual)

*(eseguito solo se CLAUDE.md indica cross-linker manual; in modalità auto è step di ingest)*

Per ogni pagina wiki: estrai `title` + `aliases`, cerca occorrenze testuali in altre pagine (case-sensitive, word boundary, fuori code/frontmatter, non già linkate). Match univoco → applica `[[<slug>]]` SOLO alla prima occorrenza per pagina. Match ambiguo → report (no fix).

---

## CHECK EURISTICI (solo report)

Questi richiedono giudizio. **Non auto-fixare mai.**

### H1 — Contraddizioni fattuali tra pagine

Per pagine che condividono tag principali, leggi e confronta i claim. Se due pagine affermano cose incompatibili sulla stessa feature/configurazione → segnala con citazioni specifiche (path + linea/sezione).

### H2 — Claim datati superati

Confronta data `updated` delle pagine con date `updated` delle src-* citate. Se una src-* più recente esiste e tratta lo stesso topic ma la pagina principale non l'ha incorporata → segnala "possibile cascade mancato".

### H3 — Conflict annotation mancanti

Pagine il cui `sources:` cita ≥2 src-* pubblicate in date molto distanti (es. >12 mesi) e che non contengono il blocco `> ⚠️ Conflitto fonti` → segnala "verificare se le fonti contraddicono e annotare esplicitamente".

### H4 — Pagine orfane

Pagine con 0 inbound link da altre pagine wiki:
```bash
find wiki -name '*.md' | while read -r f; do
  slug=$(basename "$f" .md)
  count=$(grep -rlE "\[\[$slug(\||\])" wiki/ --include="*.md" | grep -v "$f" | wc -l)
  echo "$count $f"
done | awk '$1 == 0'
```
Escludi src-* (sono spesso tecnicamente orfane ma intenzionalmente).

### H5 — Cross-topic references mancanti

Concetti citati spesso in cartelle diverse senza link reciproci. Es. termine X appare in 5 pagine `wiki/<topic-A>/` e 3 `wiki/<topic-B>/` — verifica che esistano cross-link.

### H6 — Concetti senza pagina propria

Termini ricorrenti (≥5 occorrenze totali nel corpo wiki/) NON corrispondenti a una pagina esistente. Suggerisci creazione.

```bash
grep -hoE "\b[A-Z][a-zA-Z]{3,}\b" $(find wiki -name '*.md') | sort | uniq -c | sort -rn | head -30
```

### H7 — Provenance speculativa

*(skip se la wiki non usa `^[inferred]` markers)*

Pagine con alta densità di `^[inferred]`:
```bash
find wiki -name '*.md' | while read -r f; do
  inferred=$(grep -c "\^\[inferred" "$f")
  paragraphs=$(grep -cE "^[A-Z]" "$f")
  [ "$paragraphs" -gt 0 ] && pct=$((inferred * 100 / paragraphs))
  [ "$pct" -gt 25 ] && echo "$pct% $f"
done
```
Soglia: >25% → segnala "drift verso speculazione, ingerire fonte autorevole".

### H8 — Tag duplicati semantici

Cerca coppie di tag con:
- Distanza Levenshtein bassa
- O che co-occorrono frequentemente (>50% delle pagine in cui appare uno appare anche l'altro)

Suggerisci consolidamento e marca uno come deprecato in `_meta/tags.md`.

### H9 — Tag orfani

Tag presenti in `_meta/tags.md` ma usati in ≤2 pagine. Suggerisci rimozione o assorbimento.

### H10 — raw/ non ingerito o modificato (PENDING)

```bash
# Tutti i file raw
find raw -name "*.md" -not -path "*/assets/*"
# raw_source ingeriti
grep -h "^raw_source:" wiki/src-*.md | sed 's/raw_source: *//'
```
Differenza = mai ingeriti.

Per quelli ingeriti, confronta hash. Hash diverso = "fonte modificata, ri-ingest necessario".

### H11 — Archive page con sorgenti aggiornate

*(skip se la wiki non supporta archive pages)*

Per pagine `archive: true`: confronta `archived_at` con `updated:` delle pagine in `sources:`. Se ≥1 source è stata aggiornata dopo l'archive → segnala "il ragionamento archiviato potrebbe essere obsoleto, considera nuova query".

---

## OUTPUT

```
🔧 Auto-fixed (N)
  ✓ index.md: aggiunto wiki/<topic>/<page>.md (mancante)
  ✓ index.md: marcato [MISSING] per wiki/<topic>/<old>.md
  ✓ wiki/<topic>/<page>.md: link rotto [[<slug>]] → trovato in wiki/<altro>/, fixed
  ✓ wiki/<topic>/<page>.md: aggiunto See Also reciproco con <altra>.md
  ✓ wiki/_meta/tags.md: aggiornati conteggi (<N> tag)

⚠️ Da rivedere (M)

  1. [H1] Possibile contraddizione
     wiki/<topic-A>/<a>.md:42 afferma "..."
     wiki/<topic-B>/<b>.md:18 afferma "..."
     → Verificare e aggiungere conflict annotation

  2. [H6] Concetto frequente senza pagina
     "<Term>" → 14 occorrenze in 8 pagine, nessuna pagina dedicata
     → Suggerito: wiki/concepts/<term-slug>.md

  3. [H7] Provenance speculativa
     wiki/<topic>/<page>.md → 27% paragrafi marcati ^[inferred]
     → Ingerire fonte autorevole

  4. [H8] Tag duplicati
     `<tag-a>` (3 pagine) ⊂ `<tag-b>` (12 pagine)
     → Consolidare; marcare deprecato in _meta/tags.md

  5. [H10] raw/ pendente
     raw/<path>/<file>.md → mai ingerito
     raw/<path>/<file2>.md → hash modificato (ri-ingest)
     → Lanciare /wiki-ingest senza argomenti

📊 Summary
   Auto-fixed: <N>
   Da rivedere: <M>
   Pagine totali wiki: <X>
   Pagine src-*: <Y>
   Tag attivi: <Z>

📓 Log appended.
```

## Append log.md

```markdown
## [YYYY-MM-DD] lint | <N> auto-fixed, <M> da rivedere
- Top heuristic findings: H1 (1), H6 (1), H7 (1), H8 (1), H10 (2)
```

---

## Note operative

- **CLAUDE.md mancante**: avvisa `"Questa directory non sembra una wiki. Lint richiede una wiki LLM-mantenuta. Lancia /wiki-bootstrap o /wiki-upgrade."`
- **Esegui sempre prima i check deterministici, poi gli euristici** — i deterministici possono ridurre il rumore (es. fixare un link rotto rimuove un possibile falso positivo H4)
- **Non bloccare**: anche se trovi tanto, completa tutti i check e mostra il report integrale
- **Performance**: per vault grandi (>500 pagine) considera di chiedere "lint completo o solo deterministici?"
- **Idempotenza**: due lint consecutivi sul medesimo stato devono produrre stesso output (no false positivi random)
