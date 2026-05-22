---
name: feedback-package-lock-standard
description: Always commit package-lock.json and use npm ci in CI workflows - standard adopted across all repos
metadata: 
  node_type: memory
  type: feedback
  originSessionId: e6e32652-7776-4815-bd6b-227db97b4983
---

Sempre commitar o package-lock.json nos repositórios Node.js. Usar `npm ci` + `cache: 'npm'` nos workflows de CI.

**Why:** Padrão de mercado para builds reproduzíveis. `npm install` sem lock file pode puxar versões diferentes. O usuário confirmou em 2026-05-13 após discutir a prática.

**How to apply:** Ao criar workflows de CI para repos Node.js:
- Nunca colocar `package-lock.json` no `.gitignore`
- Usar `cache: 'npm'` no `setup-node`
- Usar `npm ci` (não `npm install`)
- Incluir `package-lock.json` no `git add` do create-release (bump)
- Incluir `package-lock.json` no zip de deploy

Relacionado: [[project-branching-strategy-discussion]]
