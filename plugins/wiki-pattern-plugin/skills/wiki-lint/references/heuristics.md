# Wiki Lint — check euristici

Riferimento per la sezione "CHECK EURISTICI" di `wiki-lint/SKILL.md`.

Tutti i check qui sotto producono **solo report**, mai auto-fix. Richiedono giudizio umano.

---

## H1 — Contraddizioni fattuali tra pagine

Per pagine che condividono tag principali, leggi e confronta i claim. Se due pagine affermano cose incompatibili sulla stessa feature/configurazione → segnala con citazioni specifiche (path + linea/sezione).

## H2 — Claim datati superati

Confronta data `updated` delle pagine con date `updated` delle src-* citate. Se una src-* più recente esiste e tratta lo stesso topic ma la pagina principale non l'ha incorporata → segnala "possibile cascade mancato".

## H3 — Conflict annotation mancanti

Pagine il cui `sources:` cita ≥2 src-* pubblicate in date molto distanti (es. >12 mesi) e che non contengono il blocco `> ⚠️ Conflitto fonti` → segnala "verificare se le fonti contraddicono e annotare esplicitamente".

## H4 — Pagine orfane

Pagine con 0 inbound link da altre pagine wiki:
```bash
find wiki -name '*.md' | while read -r f; do
  slug=$(basename "$f" .md)
  count=$(grep -rlE "\[\[$slug(\||\])" wiki/ --include="*.md" | grep -v "$f" | wc -l)
  echo "$count $f"
done | awk '$1 == 0'
```
Escludi src-* (sono spesso tecnicamente orfane ma intenzionalmente).

## H5 — Cross-topic references mancanti

Concetti citati spesso in cartelle diverse senza link reciproci. Es. termine X appare in 5 pagine `wiki/<topic-A>/` e 3 `wiki/<topic-B>/` — verifica che esistano cross-link.

## H6 — Concetti senza pagina propria

Termini ricorrenti (≥5 occorrenze totali nel corpo wiki/) NON corrispondenti a una pagina esistente. Suggerisci creazione.

```bash
grep -hoE "\b[A-Z][a-zA-Z]{3,}\b" $(find wiki -name '*.md') | sort | uniq -c | sort -rn | head -30
```

## H7 — Provenance speculativa

*(skip se la wiki non usa `^[inferred]` markers)*

Pagine con alta densità di `^[inferred]`:
```bash
find wiki -name '*.md' | while read -r f; do
  inferred=$(grep -c "\^\[inferred" "$f")
  paragraphs=$(grep -cE "^[A-Z]" "$f")
  if [ "$paragraphs" -gt 0 ]; then
    pct=$((inferred * 100 / paragraphs))
    [ "$pct" -gt 25 ] && echo "$pct% $f"
  fi
done
```

Soglia: >25% → segnala "drift verso speculazione, ingerire fonte autorevole".

> **Nota**: il conteggio `paragraphs` via `grep -cE "^[A-Z]"` è euristico e può sottostimare. Se `paragraphs == 0` salta la pagina (no division). Considera questo check come segnale, non come metrica esatta.

## H8 — Tag duplicati semantici

Cerca coppie di tag con:
- Distanza Levenshtein bassa
- O che co-occorrono frequentemente (>50% delle pagine in cui appare uno appare anche l'altro)

Suggerisci consolidamento e marca uno come deprecato in `_meta/tags.md`.

## H9 — Tag orfani

Tag presenti in `_meta/tags.md` ma usati in ≤2 pagine. Suggerisci rimozione o assorbimento.

## H10 — raw/ non ingerito o modificato (PENDING)

```bash
# Tutti i file raw
find raw -name "*.md" -not -path "*/assets/*"
# raw_source ingeriti
grep -h "^raw_source:" wiki/src-*.md | sed 's/raw_source: *//'
```
Differenza = mai ingeriti.

Per quelli ingeriti, confronta hash (`sha1sum`). Hash diverso = "fonte modificata, ri-ingest necessario".

## H11 — Archive page con sorgenti aggiornate

*(skip se la wiki non supporta archive pages)*

Per pagine `archive: true`: confronta `archived_at` con `updated:` delle pagine in `sources:`. Se ≥1 source è stata aggiornata dopo l'archive → segnala "il ragionamento archiviato potrebbe essere obsoleto, considera nuova query".

## H12 — TODO/Bug stantii o malformati

*(skip se la wiki non ha modulo stato — assenza di `wiki/stato/todo.md` e `wiki/stato/bug-aperti.md`)*

Scansiona `wiki/stato/todo.md` e `wiki/stato/bug-aperti.md`:
- Voci aperte da **>30 giorni** (calcolato da `Aperto:` vs oggi) → "stantio: rivedere priorità o chiudere"
- Voci con campi obbligatori mancanti (`Modulo`, `Priorità`, `Owner`, `Stima`, `Aperto`) → "frontmatter incompleto"
- Voci con `Modulo:` non presente nella tabella moduli del CLAUDE.md → "modulo sconosciuto"
- Bug P0 aperti da **>3 giorni** → "P0 di lunga durata: verificare che sia ancora P0"

## H13 — Closure pendenti

*(skip se la wiki non ha modulo stato)*

Per ogni nota raw/inbox già ingerita (presente come `src-*.md` o registrata in `log.md` con tipo `ingest`/`inbox-process`) che dichiara nel frontmatter `chiude_todo:` o `chiude_bug:`:
- Verifica che gli ID elencati NON siano più presenti in `todo.md` / `bug-aperti.md`.
- Se un ID dichiarato chiuso è ancora aperto → "closure non applicata: re-ingest la nota o rimuovi manualmente la voce".
