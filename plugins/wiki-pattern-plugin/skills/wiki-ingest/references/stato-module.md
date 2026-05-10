# Modulo stato — operazioni di ingest

Riferimento per lo Step 11 di `wiki-ingest/SKILL.md`.

**Esegui questo step solo se** la wiki ha sezione "Modulo stato" nel `CLAUDE.md` (feature flag `stato_pages: true`) E i file `wiki/stato/todo.md` e `wiki/stato/bug-aperti.md` esistono. Altrimenti skip.

Lo stato si aggiorna **ad ogni ingest** (raw e inbox) leggendo la nota in due modi: sezioni esplicite `## TODO emersi` / `## Bug aperti emersi` per nuove voci, frontmatter `chiude_*` / `aggiorna_*` per modifiche a voci esistenti.

---

## 11.a — Sezioni esplicite della nota

Cerca nella nota due sezioni opzionali:

```markdown
## TODO emersi
- [P1] [auth] Migrare login a OAuth2 — Owner: rocco — Stima: M
  Note: dipende da rotazione chiavi
- [P2] [infra] Audit log retention policy

## Bug aperti emersi
- [P0] [auth] Login lento >5s su tenant XL — Owner: rocco — Stima: M
  Sintomo: timeout su GET /me dopo OAuth
  Note: probabile N+1 nelle policy
```

Sintassi riga: `- [<priorità>] [<modulo>] <titolo> — Owner: <owner> — Stima: <S|M|L>`. Le righe successive (indentate) sono campi opzionali (`Sintomo:`, `Note:`).

**Schema modulo**: i moduli ammessi sono quelli elencati in CLAUDE.md della wiki (sezione "Moduli del progetto" o equivalente). Modulo non riconosciuto → warning, voce comunque aggiunta.

## 11.b — Frontmatter di chiusura

Leggi questi campi opzionali dal frontmatter della nota raw:

```yaml
chiude_todo: [TODO-042, TODO-043]
chiude_bug: [BUG-017]
aggiorna_todo:
  - id: TODO-040
    priorita: P0
    note: "escalato dopo incidente prod"
aggiorna_bug:
  - id: BUG-015
    owner: maria
```

## 11.c — Operazioni su `wiki/stato/todo.md`

1. **Read** del file. Estrai il `next_id:` dal frontmatter.
2. Per ogni voce nella sezione `## TODO emersi`: assegna ID `TODO-<next_id>`, incrementa contatore, aggiungi blocco strutturato:
   ```markdown
   ### TODO-NNN — <titolo>
   - Modulo: <modulo>
   - Priorità: <P0|P1|P2>
   - Owner: <owner|unassigned>
   - Stima: <S|M|L>
   - Aperto: <data nota>
   - Fonte: [[<nome-nota-raw>]]
   - Note: <note se presenti>
   ```
3. Per ogni ID in `chiude_todo`: rimuovi il blocco corrispondente. Se non trovato: warning, non fatale.
4. Per ogni entry di `aggiorna_todo`: aggiorna i campi indicati nel blocco esistente. Se non trovato: warning.
5. Aggiorna `next_id:` e `data_aggiornamento:` nel frontmatter.
6. **Write** del file.

## 11.d — Operazioni su `wiki/stato/bug-aperti.md`

Stesso flusso di 11.c con prefisso `BUG-`. Schema voce:

```markdown
### BUG-NNN — <titolo>
- Modulo: <modulo>
- Priorità: <P0|P1|P2>
- Owner: <owner|unassigned>
- Stima: <S|M|L>
- Aperto: <data nota>
- Fonte: [[<nome-nota-raw>]]
- Sintomo: <sintomo se presente>
- Note: <note se presenti>
```

## 11.e — Riassunto in log.md

Nel blocco di log dello Step 10 (sezione principale), aggiungi una riga `Stato:` se almeno un'operazione su stato pages è avvenuta:

```markdown
- Stato: aggiunti TODO-042, TODO-043; chiusi BUG-015; aggiornato TODO-040 (P0)
```

## 11.f — Edge case stato

- **`next_id` mancante** in todo.md/bug-aperti.md: assumi 1, riscrivi frontmatter.
- **ID duplicato** in apertura: skip + warning (rispetta sempre la sequenzialità di `next_id`).
- **Sezione `## TODO emersi` presente ma vuota**: skip silenzioso.
- **Modulo non in elenco CLAUDE.md**: aggiungi comunque, warning in output.
