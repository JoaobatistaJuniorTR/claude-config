---
name: Checkbox Align With Text-Field
description: Padrao para alinhar saf-checkbox verticalmente com input de saf-text-field em saf-layout-grid
type: feedback
---

Para alinhar um saf-checkbox ao centro do input de um saf-text-field na mesma linha de grid:

```html
<saf-layout-grid-item xs="12" md="4" style="justify-content: end;">
  <div class="checkbox-align-with-input">
    <saf-checkbox>Label</saf-checkbox>
  </div>
</saf-layout-grid-item>
```

```scss
.checkbox-align-with-input {
  display: flex;
  align-items: center;
  height: var(--saf-spacing-10); // 40px = altura do input em density standard
}
```

**Why:** O saf-text-field tem label acima do input (~32px). `align-self: center` centraliza na row inteira (fica na altura do label). `align-self: end` alinha embaixo demais. A solução correta: `justify-content: end` no grid-item (flex-column empurra para baixo) + wrapper com altura fixa igual ao input (40px = --saf-spacing-10) + align-items: center.

**How to apply:** Não existe solução oficial do Saffron para este caso. Não existe saf-form-field ou componente wrapper de formulário. Para density compact, a altura do input é 32px.
