---
name: Saf-Dialog Width
description: Custom property correta para largura do saf-dialog e --saf-dialog-minWidth (camelCase), nao kebab-case
type: feedback
---

Para controlar a largura do saf-dialog, usar `--saf-dialog-minWidth` (camelCase). NÃO usar `--saf-dialog-max-width` ou `--saf-dialog-min-width` (kebab-case) — essas variáveis não existem.

O max-width do dialog é `50vw` hardcoded no Shadow DOM (media query). Para forçar largura maior, setar `--saf-dialog-minWidth` no valor desejado (ex: 850px).

**Why:** Tentamos `--saf-dialog-max-width: 850px` e `--saf-dialog-min-width: 700px` — nenhum efeito. O saf-dialog usa camelCase nas custom properties internas.

**How to apply:** Inspecionar dialog.styles.js para confirmar nomes reais de custom properties antes de usar.
