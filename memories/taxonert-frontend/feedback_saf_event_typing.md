---
name: Saffron Event Typing
description: Eventos de web components saf-* no template Angular devem usar any, nao CustomEvent
type: feedback
---

Eventos de web components saf-* (como saf-pagination change, items-per-page-change) passam `$event` como `Event` generico no template Angular, nao como `CustomEvent<T>`. Tipar o parametro do handler como `CustomEvent<number>` quebra o build.

**Why:** O auditor Angular (B4) tentou substituir `any` por `CustomEvent<number> | number` nos handlers setPageIndex/setItemsPerPage, causando erro de build: "Argument of type 'Event' is not assignable to parameter of type 'CustomEvent<number>'".

**How to apply:** Para event handlers de web components saf-* no template, manter `event: any` no .ts. Essa e a excecao permitida na regra B4 do angular-auditor. O auditor NAO deve tipar esses handlers — o tipo correto nao e exportado pelo Saffron.
