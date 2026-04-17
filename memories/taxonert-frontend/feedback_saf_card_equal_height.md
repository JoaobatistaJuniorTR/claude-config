---
name: Saf-Card Equal Height
description: Para cards com mesma altura em grid, usar ::part(wrapper) com height 100%
type: feedback
---

Para que saf-cards em um grid tenham a mesma altura:

```scss
.profile-card {
  height: 100%;

  &::part(wrapper) {
    height: 100%;
  }
}
```

**Why:** O saf-card tem :host display:block e .wrapper interno com display:flex. height:100% no :host não é suficiente — o .wrapper interno não herda. Como o saf-card expõe `part="wrapper"`, podemos estilizar via ::part(wrapper).

**How to apply:** Sempre que precisar de equal-height cards, adicionar ::part(wrapper) { height: 100% }. Para áreas de texto variável, adicionar min-height na classe de descrição.
