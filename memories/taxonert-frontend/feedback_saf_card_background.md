---
name: feedback_saf_card_background
description: Padrão para adicionar imagem de fundo decorativa no canto superior direito de saf-cards
type: feedback
---

Usar imagem SVG decorativa (`saf-card-background.svg`) no canto superior direito de todos os cards (`saf-card`). O shadow DOM do saf-card impede posicionamento direto — a solução é um wrapper interno com margem negativa.

**Why:** O `saf-card` tem shadow DOM com `.wrapper` que aplica `background-color`, `border`, `overflow: hidden` e padding interno (`--saf-card-container-padding`, `--saf-card-content-margin`). CSS externo não consegue posicionar elementos relativos à borda do card. Tentativas anteriores falharam: CSS `background-image` no host (shadow DOM bloqueia), wrapper externo (card tem background próprio que cobre a imagem), wrapper interno sem compensação (imagem fica no conteúdo, não na borda).

**How to apply:**

1. **SVG em `projects/rt/public/saf-card-background.svg`** — angular.json serve assets de `projects/rt/public/`, URL = `/saf-card-background.svg`

2. **NUNCA usar `heading-level` nem `slot="heading"` no saf-card quando usar marca d'água.** O `slot="heading"` muda a estrutura interna do shadow DOM e a compensação de margem negativa não funciona. O título deve ser um `saf-text` dentro do conteúdo do card (com `color: var(--saf-color-text-subtle)` para cinza).

3. **HTML — wrapper dentro do saf-card, ANTES do conteúdo:**
   ```html
   <saf-card class="meu-card">
     <div class="card-bg-wrapper card-bg-wrapper--padrao">
       <div class="card-bg-image"></div>
     </div>
     <saf-text appearance="heading-lg" class="card-titulo">Título do card</saf-text>
     <!-- conteúdo do card aqui -->
   </saf-card>
   ```

4. **SCSS — classes base (copiar para cada componente que usar):**
   ```scss
   .card-titulo {
     color: var(--saf-color-text-subtle);
   }

   .card-bg-wrapper {
     position: relative;
     height: 0;
     overflow: visible;
   }

   // Margem negativa compensa padding interno do shadow DOM do saf-card
   // spacing-5 (container-padding) + spacing-4 (content-margin) + spacing-2 (header-margin)
   // margin-bottom devolve o espaço ao conteúdo abaixo
   .card-bg-wrapper--padrao {
     margin: calc(-1 * var(--saf-spacing-5) - var(--saf-spacing-4) - var(--saf-spacing-2))
             calc(-1 * var(--saf-spacing-5))
             var(--saf-spacing-2);
   }

   .card-bg-image {
     position: absolute;
     top: 0;
     right: 0;
     width: 162px;
     height: 161px;
     background-image: url('/saf-card-background.svg');
     background-repeat: no-repeat;
     background-size: contain;
     pointer-events: none;
     z-index: 0;
     opacity: 0.5;
   }
   ```

5. **Por que funciona:** O wrapper tem `height: 0` (não ocupa espaço no layout). A margem negativa puxa o wrapper para cima/direita, compensando o padding do shadow DOM. O `card-bg-image` é `position: absolute` relativo ao wrapper, ficando alinhado à borda do card. O `z-index: 0` e `opacity: 0.3` garantem que a imagem fique por trás do conteúdo como marca d'água sutil. O `margin-bottom` positivo devolve o espaço ao conteúdo que vem depois.

6. **Essa mesma margem negativa funciona para cards com `--saf-card-container-padding: var(--saf-spacing-5)` E para cards com `--saf-card-container-padding: 0`** — o shadow DOM mantém espaçamentos internos mesmo com custom properties zeradas.
