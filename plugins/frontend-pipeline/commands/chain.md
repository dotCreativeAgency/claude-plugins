---
name: chain
description: "Activate mandatory agent chain enforcement for frontend pipeline"
args:
  - name: plan
    description: "The implementation plan or task to execute through the agent chain"
    required: true
---

# AGENT CHAIN ENFORCEMENT MODE

## REGOLA ASSOLUTA

Quando questo comando è attivo, è **VIETATO** implementare codice React direttamente. OGNI task di implementazione frontend DEVE passare attraverso la catena di agenti specializzati usando il tool `Task`.

## CATENA OBBLIGATORIA

Per ogni componente/task del piano fornito dall'utente, esegui questa sequenza:

```
1. frontend-architect   -> Crea il componente (design + implementazione)
2. frontend-reviewer    -> Code review con score numerico (OBBLIGATORIO)
3. frontend-debugger    -> SE score < 80 OPPURE issue CRITICAL/HIGH -> fix automatico
4. frontend-refactorer  -> SE LOC > 150 OPPURE maintainability < 15/25 -> decomposizione
5. frontend-reviewer    -> Ri-validazione dopo debugger/refactorer (max 2 cicli)
```

## REGOLE DI ESECUZIONE

### Lancio agenti
- Usa SEMPRE il tool `Task` con `subagent_type` corrispondente al nome dell'agente
- Fornisci un prompt DETTAGLIATO con tutto il contesto necessario (file coinvolti, requisiti, output dell'agente precedente)
- NON procedere alla fase successiva finché l'agente corrente non ha completato

### Cosa NON fare MAI
- Leggere file e implementare componenti React direttamente senza agenti
- Saltare il reviewer dopo che l'architect ha completato
- Non lanciare il debugger quando il reviewer dà score < 80
- Riassumere cosa "faresti" invece di lanciare effettivamente l'agente
- Implementare "task semplici" direttamente perché sembrano banali
- Ignorare LOC > 150 senza lanciare il refactorer

### Post-review automatica
- Score < 80 -> lancia `frontend-debugger` automaticamente
- LOC > 150 o maintainability < 15/25 -> lancia `frontend-refactorer` automaticamente
- Score >= 80 ma con issue MEDIUM -> chiedi all'utente se vuole il debugger
- Dopo debugger/refactorer -> rilancia `frontend-reviewer` per verifica (max 2 cicli)

### Gestione fasi multiple
- Completa l'INTERA catena per un componente prima di passare al successivo
- Tra una fase e l'altra, riporta all'utente: fase completata, score, prossima fase
- Se una fase fallisce, NON procedere: riporta l'errore e attendi istruzioni

## FORMATO OUTPUT TRA FASI

Dopo ogni agente completato, riporta:

```
Fase N - [nome componente]
   Agente: [nome agente] -> Completato
   Risultato: [breve sintesi]
   Score: [se reviewer: N/100]
   Prossimo: [nome prossimo agente o "Componente N+1"]
```

## PIANO DELL'UTENTE

Processa il seguente piano rispettando TUTTE le regole sopra:

$ARGUMENTS
