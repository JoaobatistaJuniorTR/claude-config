---
name: feedback_no-stdin-redirect-gh
description: Never use @- (stdin redirect) with gh CLI on PowerShell — pass content inline via --notes or use Bash heredoc
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 39bbd102-7bc1-4104-85a5-83eacaa8de31
---

Não usar `@-` ou `<<'EOF'` com `gh release edit --notes` no PowerShell — o shell não interpreta corretamente e o conteúdo vira literal `@-`.

**Why:** Já aconteceu duas vezes. PowerShell não suporta heredoc estilo Bash. O `@-` (leia do stdin) também não funciona como esperado no contexto do Bash tool quando combinado com `gh`.

**How to apply:** Sempre passar o conteúdo diretamente como string no `--notes '...'` usando aspas simples (que suportam multilinha no Bash). Alternativa: escrever em arquivo temporário e usar `--notes-file`.
