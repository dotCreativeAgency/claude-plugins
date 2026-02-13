# dotCreativeAgency Claude Plugins

Marketplace di plugin per Claude Code.

## Come usare

```bash
# Aggiungi il marketplace
/plugin marketplace add dotCreativeAgency/claude-plugins

# Esplora i plugin disponibili nella tab Discover
/plugin
```

## Plugin disponibili

| Plugin | Versione | Descrizione |
|--------|----------|-------------|
| [frontend-pipeline](./plugins/frontend-pipeline) | 1.2.0 | Pipeline automatizzata per lo sviluppo frontend React + MUI + TypeScript con 5 agenti specializzati, chain enforcement obbligatoria e comando /chain |

## Contribuire

Questo è un monorepo che contiene tutti i plugin del marketplace.

### Aggiungere un nuovo plugin

1. Crea una directory in `plugins/nome-plugin/`
2. Aggiungi `.claude-plugin/plugin.json` con i metadati del plugin
3. Implementa agents/skills/commands secondo le necessità
4. Aggiungi il plugin in `.claude-plugin/marketplace.json`
5. Aggiorna questo README con il link al plugin

### Modificare un plugin esistente

I plugin sono contenuti in `plugins/`. Puoi modificarli direttamente e fare commit atomici che includono sia modifiche al plugin che al marketplace.

## Licenza

MIT
