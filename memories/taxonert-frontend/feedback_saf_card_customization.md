---
name: feedback_saf_card_customization
description: Como customizar padding e margin internos do saf-card via CSS custom properties
type: feedback
---

O `saf-card` tem shadow DOM com padding interno significativo que não pode ser estilizado diretamente. Usar CSS custom properties expostas pelo componente para ajustar.

**Why:** Títulos dentro de `saf-card` ficavam muito distantes do topo do quadro. O default `--saf-card-container-padding` é `--saf-spacing-6` (24px). Reduzir para `--saf-spacing-5` resolveu para o card de inconsistências. Para Detalhes, tentamos `0` mas não surtiu efeito total — pode haver outros espaçamentos internos no shadow DOM. Investigar via DevTools quando necessário.

**How to apply:**

1. **Custom properties disponíveis** (encontradas no source `card.scss`):
   - `--saf-card-container-padding` — padding geral do container interno (default: `--saf-spacing-6`)
   - `--saf-card-content-margin` — margin do conteúdo slotted (default: `--saf-spacing-4 0`)
   - `--saf-card-media-padding` — padding da área de media
   - `--saf-card-heading-font` — font do heading
   - `--saf-card-badge-right` — posição do badge

2. **Exemplo de uso:**
   ```scss
   .card-inconsistencias {
     --saf-card-container-padding: var(--saf-spacing-5);
   }
   .card-detalhes {
     --saf-card-container-padding: 0;
     --saf-card-content-margin: 0;
   }
   ```

3. **Se custom properties não forem suficientes**, o shadow DOM pode ter espaçamentos adicionais. Verificar no DevTools do navegador inspecionando o shadow root do `saf-card`.

4. **Source de referência:** `C:\repositories\saffron_design_system\core-packages\core-components\src\components\card\card.scss`
