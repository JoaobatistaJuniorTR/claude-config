---
name: feedback-ado-comments-html
description: Comentários no ADO devem usar HTML com <br> entre parágrafos para espaçamento visual adequado
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 7c82748e-fa93-46e6-8790-f29d8ca5693d
---

Comentários no Azure DevOps devem ser escritos em HTML (format: Html), não em Markdown puro.
Usar `<br>` entre os `<p>` para dar espaçamento visual entre seções.

**Why:** Markdown com `\n` literais não renderiza corretamente no ADO — aparece tudo numa linha só. Mesmo usando HTML, sem `<br>` entre parágrafos o texto fica colado.

**How to apply:** Ao criar/editar comentários no ADO via MCP, sempre usar format Html com tags `<h2>`, `<p>`, `<b>`, `<code>`, `<ul><li>` e `<br>` entre blocos para respiro visual.
