---
name: wiki-briefing
description: "Briefing operativo di una wiki LLM-mantenuta del pattern Karpathy esteso: fatto di recente, bug aperti, TODO per priorità. Solo nelle wiki con feature flag stato_pages attivo (file wiki/stato/todo.md e wiki/stato/bug-aperti.md presenti). Triggers: '/wiki-briefing', 'buongiorno wiki', 'buonpomeriggio wiki', 'buonasera wiki', 'ciao wiki', 'briefing wiki', 'stato wiki', 'cosa abbiamo fatto', 'cosa è rimasto in sospeso', 'cosa dobbiamo fare'."
---

# Wiki Briefing

Genera un briefing operativo della wiki LLM-mantenuta della directory di lavoro corrente.

**Pre-requisito**: la directory deve contenere `CLAUDE.md` con il pattern wiki E avere il feature flag `stato_pages: true` (verifica la presenza dei file `wiki/stato/todo.md` e `wiki/stato/bug-aperti.md`). Se il flag è off → rispondi `"Questa wiki non ha il modulo 'stato' attivo. Lancia /wiki-upgrade per abilitarlo."`

## Modalità di invocazione

| Input | Comportamento |
|---|---|
| `/wiki-briefing` | Briefing standard, finestra 7 giorni |
| `/wiki-briefing 14g` (o `Ng`) | Briefing con finestra custom in giorni |
| `buongiorno wiki` / `ciao wiki` / etc | Briefing standard |
| `cosa abbiamo fatto?` | Solo sezione "Fatto di recente" |
| `cosa è rimasto in sospeso?` / `cosa dobbiamo fare?` | Solo sezioni "Bug aperti" + "TODO" |

## Step 1 — Verifica precondizioni (HARD GUARD)

I trigger su saluti naturali (`buongiorno wiki`, `ciao wiki`, ecc.) sono potenti ma generici. Prima di qualsiasi altra azione:

```bash
test -f CLAUDE.md && test -f wiki/stato/todo.md && test -f wiki/stato/bug-aperti.md
```

**Se la cwd NON è una wiki LLM-mantenuta** (manca `CLAUDE.md` con il pattern wiki, o mancano i file di stato):

- Se l'utente ha invocato esplicitamente `/wiki-briefing` o ha scritto un trigger che contiene il token `wiki` (`buongiorno wiki`, `briefing wiki`, `stato wiki`, ecc.) → rispondi con il messaggio di Edge case appropriato.
- Se l'utente ha scritto un trigger generico **senza** la parola "wiki" (es. `cosa abbiamo fatto?`, `cosa dobbiamo fare?`) → **non attivare la skill**, defer al normale flusso conversazionale.

In altre parole: la skill si attiva solo se (a) la cwd è una wiki valida con `stato_pages` attivo OPPURE (b) l'utente ha menzionato esplicitamente "wiki" o usato lo slash command. Mai attivarsi silenziosamente in un progetto non-wiki su un saluto generico.

## Step 2 — Determina il saluto in base all'ora locale

Leggi l'ora corrente (Italia, fuso `Europe/Rome`):
- 05:00–11:59 → "🌅 Buongiorno"
- 12:00–17:59 → "☀️ Buon pomeriggio"
- 18:00–22:59 → "🌆 Buonasera"
- 23:00–04:59 → "🌙 Ancora qui"

Se l'utente ha scritto un saluto specifico (`buongiorno`/`buonpomeriggio`/`buonasera`/`ciao`), usa quello come override.

## Step 3 — Carica i dati

Tre `Read` in parallelo:
1. `wiki/stato/todo.md` — parsa i blocchi `### TODO-NNN`
2. `wiki/stato/bug-aperti.md` — parsa i blocchi `### BUG-NNN`
3. `log.md` — leggi le entry con data ≥ (oggi - finestra). Default finestra: 7 giorni. Override: argomento `Ng`.

## Step 4 — Calcola riassuntino numerico

Conta:
- TODO totali aperti, suddivisi per priorità (P0/P1/P2)
- Bug totali aperti, suddivisi per priorità
- Numero entry `log.md` nella finestra

## Step 5 — Componi output

```markdown
{{SALUTO}}. Ecco lo stato di {{DOMINIO}}.

📊 {{N_TODO}} TODO aperti ({{N_P0}}×P0, {{N_P1}}×P1, {{N_P2}}×P2) · {{N_BUG}} bug aperti ({{N_P0_BUG}}×P0, ...) · {{N_FATTO}} modifiche negli ultimi {{N}} giorni

## ✅ Fatto di recente ({{N}}g)

<max 10 entry da log.md, ordine cronologico inverso>
- **YYYY-MM-DD** — *tipo* — Titolo (pagine: [[link1]], [[link2]])
- ...
<se overflow: "...e altre N modifiche, vedi log.md">

## 🐞 Bug aperti

<TUTTI i bug, ordinati per priorità ASC poi data apertura ASC>
- **BUG-NNN** [P0] *(modulo)* — Titolo
  Sintomo: ... · Owner: X · Aperto: YYYY-MM-DD · Stima: M
  Note: ...
- ...

## 📋 TODO

<TUTTI i P0 e P1, ordinati per priorità ASC poi data apertura ASC>
- **TODO-NNN** [P1] *(modulo)* — Titolo
  Owner: X · Aperto: YYYY-MM-DD · Stima: S
  Note: ...
- ...

<se ci sono P2: "+{{N_P2}} TODO P2 — chiedi `/wiki-query todo P2` per dettagli">
```

## Step 6 — Modalità ridotte

Se l'utente ha chiesto solo "cosa abbiamo fatto" → mostra solo sezione "Fatto di recente" + riga di riassunto numerico in cima.

Se ha chiesto "cosa è rimasto in sospeso" / "cosa dobbiamo fare" → mostra solo Bug + TODO + riassunto numerico.

In entrambi i casi mantieni il saluto orario in cima.

## Step 7 — Suggerimenti contestuali (opzionali, in coda)

Se ricorrono pattern utili, suggerisci azioni:
- **Bug P0 aperti da >7 giorni**: `"⚠️ BUG-NNN aperto da N giorni — vuoi escalarlo?"`
- **TODO senza Owner > 3**: `"💡 Hai N TODO senza owner. Vuoi assegnarli?"`
- **Nessun ingest negli ultimi 7g**: `"📥 Nessun ingest recente — hai note in raw/ o inbox/ da processare?"` (verifica con find)
- **Closure pending**: nota raw recente con `chiude_todo:` ma TODO ancora in lista → `"⚠️ La nota X dichiara di chiudere TODO-NNN ma è ancora aperto. Re-ingest?"`

Massimo 2 suggerimenti per briefing, scegli i più rilevanti.

---

## Output di esempio

```
🌅 Buongiorno. Ecco lo stato di OpenTicketService.

📊 4 TODO aperti (1×P0, 2×P1, 1×P2) · 2 bug aperti (1×P0, 1×P1) · 6 modifiche negli ultimi 7 giorni

## ✅ Fatto di recente (7g)

- **2026-05-10** — *ingest* — AWS architettura overview (pagine: [[aws-infrastruttura]], [[aws-deploy]])
- **2026-05-08** — *feature* — Deploy SPA facilegestire-rete on AWS staging (pagine: [[aws-deploy]])
- **2026-05-07** — *refactor* — Seed prod/dev split (pagine: [[seeders-prod]])

## 🐞 Bug aperti

- **BUG-001** [P0] *(auth)* — Login lento >5s su tenant XL
  Sintomo: timeout su GET /me dopo OAuth · Owner: rocco · Aperto: 2026-05-09 · Stima: M
  Note: probabile N+1 nelle policy
- **BUG-002** [P1] *(notification)* — Web push non arriva su Safari iOS

## 📋 TODO

- **TODO-001** [P0] *(infra)* — Configurare backup S3 cross-region
  Owner: rocco · Aperto: 2026-05-10 · Stima: M
  Note: requisito GDPR
- **TODO-002** [P1] *(auth)* — Migrare login a OAuth2

+1 TODO P2 — chiedi `/wiki-query todo P2` per dettagli

⚠️ BUG-001 aperto da 1 giorno (P0) — verifica priorità.
```

---

## Edge case

- **Wiki senza modulo stato**: `"Questa wiki non ha il modulo 'stato' attivo. Lancia /wiki-upgrade per abilitare wiki/stato/."`
- **`todo.md` o `bug-aperti.md` vuoti**: mostra sezione con `"_Nessun TODO aperto._"` / `"_Nessun bug aperto._"`. Output positivo, non errore.
- **`log.md` vuoto o senza entry recenti**: `"_Nessuna modifica registrata negli ultimi {{N}} giorni._"`
- **Frontmatter di una voce malformato**: includi comunque la voce, marca con `⚠️ frontmatter incompleto` e suggerisci `/wiki-lint`.
- **CLAUDE.md mancante**: `"Questa directory non sembra una wiki LLM-mantenuta. Lancia /wiki-bootstrap per crearne una."`

---

## Note implementative

- Non modificare nessun file durante il briefing. È **solo lettura**.
- Non aggiornare `log.md` per il briefing stesso (sarebbe rumore).
- Le emoji nel briefing sono **intenzionali** — è output utente "vivo", non contenuto wiki.
- Tutti gli URL/link `[[wikilink]]` puntano a pagine compilate, mai a `raw/`.
