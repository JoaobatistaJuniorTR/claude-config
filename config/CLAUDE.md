# Global CLAUDE.md

Instruções globais que se aplicam a TODOS os projetos.

## Idioma e Comunicação

- **Sempre comunicar em português brasileiro** (pt-BR)
- Código e nomes técnicos em inglês
- Comentários de código em inglês
- Mensagens de commit, PRs e comunicação com o usuário em pt-BR

## Estilo de Pull Requests

**OBRIGATÓRIO**: Todo PR criado em qualquer repositório DEVE seguir este estilo de descrição. Não gere PRs genéricos ou sem personalidade.

### Tom e Linguagem
- **Português brasileiro**, informal e com humor técnico
- Narrar as mudanças como uma **história** — o que estava errado, por que doía, o que mudou
- Usar analogias e metáforas do dia a dia dev (ex: "Frankenstein", "gambiarra", "festa que ninguém pediu")
- Fechar com uma **citação irônica/filosófica** relacionada ao tema do PR

### Estrutura Obrigatória

```markdown
## 🔧 O que rolou aqui
Parágrafo narrativo explicando o PROBLEMA de forma dramática e divertida.
Contextualize o cenário — o que funcionava, onde quebrou, por que era um problema.

## 🪦 R.I.P. — O que foi enterrado
Lista com bullet points do que foi REMOVIDO/SUBSTITUÍDO.
Cada item com um comentário irônico curto sobre por que mereceu morrer.

## 🚀 O que entrou no lugar
Tabela comparativa "Antes (☠️) vs Depois (✨)" com as mudanças principais.
| Antes (☠️) | Depois (✨) |
|-----------|------------|
| Coisa ruim antiga | Coisa boa nova |

## 📋 Commits
Lista numerada dos commits com hash curto e descrição.

## 🧪 Como testar
Passos numerados para validar as mudanças manualmente.

## 💀 Risco
Avaliação honesta do risco (Baixo/Médio/Alto) com justificativa curta.

---
> *"Citação irônica/filosófica relacionada ao tema"* — Autor real ou fictício
```

### Regras Extras
- Se o PR for apenas 1 commit simples, pode simplificar mas MANTER o tom narrativo
- Adaptar os emojis das seções ao contexto (feat, fix, refactor, etc.)
- A seção "R.I.P." só aparece quando algo foi removido/substituído
- SEMPRE incluir a assinatura `🤖 Generated with [Claude Code](https://claude.com/claude-code)` no final

## Estilo de GitHub Releases

**OBRIGATÓRIO**: Toda Release criada via `gh release create` DEVE seguir este template. O tom é o mesmo dos PRs (narrativo, com humor técnico), mas a estrutura é orientada a **impacto por versão**, não por commit individual.

### Estrutura Obrigatória

```markdown
## 🚀 [Título curto e dramático da versão]

[Parágrafo narrativo: o que essa versão representa.
Qual era o estado do mundo antes, o que mudou, por que importa.
Tom: changelog com personalidade, não relatório corporativo.]

## ✨ Destaques

[Bullet points das mudanças mais importantes — agrupadas por FEATURE,
não por commit. Cada item é uma frase que QA/PM entende sem ler código.]

## 🐛 Correções

[Bullet points dos bugs corrigidos. Referência ao ADO/issue quando existir.]

## 🗑️ Removido

[O que saiu — só aparece quando algo foi removido/substituído.
Comentário curto e irônico.]

## 🎯 Áreas impactadas

[Lista das telas/módulos que mudaram — guia rápido pra QA saber
o que retestar. Formato: módulo → o que mudou.]

## 💀 Risco
[Baixo/Médio/Alto] — [justificativa em uma frase]

## 📋 PRs incluídos
[Lista dos PRs com número e título — link para rastreabilidade]

---
> *"Citação irônica relacionada ao tema da versão"* — Autor real ou fictício

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

### Diferenças do PR
- **"Áreas impactadas"** em vez de "Como testar" — QA-friendly
- **"PRs incluídos"** em vez de commits individuais — rastreabilidade sem poluição
- **"Destaques"** agrupa por feature, não por commit
- Sem tabela "Antes vs Depois" — funciona melhor no PR onde a mudança é específica
- A seção "🗑️ Removido" só aparece quando algo foi removido/substituído
- SEMPRE incluir a assinatura `🤖 Generated with [Claude Code](https://claude.com/claude-code)` no final

## Padrão de Localização de Agentes

**OBRIGATÓRIO**: Todo agente Claude vive em local fixo, com estrutura padronizada.

### Caminho e nomenclatura
- **Caminho do arquivo:** `~/.claude/agents/<nome>.md`
- **Nome do arquivo:** sempre **lowercase**, sem espaços, sem caracteres especiais (use hífen pra separar)
- **Frontmatter obrigatório:** YAML no topo com `name` e `description`. Opcional: `tools`, `model`
- **Referência canônica:** `~/.claude/agents/sincerao.md` é o template-padrão de agente bem-formado

### Memória persistente do agente (quando aplicável)
Se o agente precisa lembrar contexto entre sessões:
- **Caminho:** `~/.claude/agents/<nome>-memory/`
- **Estrutura recomendada:** subpastas por tipo (ex: `decisions/`, `findings/`, `schemas/`)
- **Anti-falsa-memória:** todo arquivo de memória com timestamp. Dado com mais de 30 dias dispara aviso de "pode estar defasado" e o agente revalida antes de fechar recomendação

### Documentação interna do agente
- Convenções específicas do agente (allowlists, gatilhos de modo, formatos de output, políticas de segurança) ficam **DENTRO do próprio `<nome>.md`** — nunca em README separado, nunca em comentário de código
- Razão: convenção órfã morre. Daqui a 3 meses ninguém lembra onde estava

### Família de agentes (múltiplos agentes relacionados)
- Quando vários agentes formarem uma família, use prefixo compartilhado (ex: `saffron-*`, `slon-*`)
- Documentação compartilhada da família vai em `~/.claude/agents/<FAMILIA>-TEMPLATE.md` — **CAPS proposital** pra distinguir visualmente de agente individual
- Exemplo existente: `AGENTS-SAFFRON-TEMPLATE.md` é doc da família Saffron, não é agente

### Ao criar novo agente
1. Confirme com o usuário se é agente mesmo (vs skill, vs slash command)
2. Crie em `~/.claude/agents/<nome>.md` (lowercase)
3. Documente todas as convenções DENTRO do arquivo
4. Se precisar de memória, crie `~/.claude/agents/<nome>-memory/` junto

## Agente Sincerão — Ativação Automática

**OBRIGATÓRIO**: Sempre que o usuário estiver discutindo **ideias, propostas, planos, features novas, brainstorming, decisões estratégicas, arquitetura de solução, ou qualquer conceito novo**, você DEVE automaticamente consultar o agente `sincerao` antes de dar sua resposta final.

### Detecção de Contexto
Ative o Sincerão quando detectar qualquer um desses sinais:
- Frases como "e se a gente...", "tive uma ideia", "estou pensando em...", "que tal...", "minha proposta é...", "quero criar...", "quero fazer...", "seria legal se..."
- Discussões sobre novas features, produtos, abordagens, arquiteturas
- Brainstorming, ideação, planejamento estratégico
- Qualquer momento em que o usuário apresenta algo novo para validação
- Decisões técnicas ou de produto que envolvem trade-offs

### Como Consultar
1. Use a ferramenta `Agent` para spawnar o agente `sincerao` (subagent_type não precisa, use o agent file)
2. Passe para ele um resumo claro da ideia/proposta do usuário com todo o contexto relevante
3. Receba o parecer dele
4. Se o parecer for "PRECISA DE AJUSTE" ou "REPROVADO": refine a ideia incorporando as críticas e envie de volta ao Sincerão
5. Continue o loop (máximo 7 iterações) até receber "APROVADO" ou até estabilizar em consenso
6. NÃO peça permissão ao usuário — faça automaticamente

### Loop Iterativo
- O Sincerão responde com status: APROVADO, PRECISA DE AJUSTE, ou REPROVADO
- Se não for APROVADO: incorpore o feedback, refine a proposta, e consulte novamente
- Máximo de 7 rodadas de iteração
- Se após 7 rodadas não houver consenso, apresente o estado final com as divergências

### Relatório ao Usuário
Após a consulta (e possíveis iterações), apresente ao usuário:

```markdown
---
### 🎭 Parecer do Sincerão
**Rodadas de análise:** X iteração(ões)
**Status final:** APROVADO / PRECISA DE AJUSTE / REPROVADO

**Resumo da crítica:**
[Pontos principais que o Sincerão levantou]

**O que mudou por causa dele:**
[Como a resposta/proposta foi ajustada com base nas críticas]

**Veredito final:**
[Conclusão consolidada]
---
```

### Regras Importantes
- NUNCA pule a consulta ao Sincerão em contextos de ideação — isso é obrigatório
- NUNCA peça permissão ao usuário para consultar — faça silenciosamente
- SEMPRE mostre o relatório de parecer ao final para transparência
- O Sincerão NÃO é ativado automaticamente para tarefas puramente operacionais (fix de bug, refactor simples, commit, PR) — mas PODE ser chamado manualmente nelas

### Invocação Manual
O usuário pode chamar o Sincerão a qualquer momento com frases como:
- "chama o sincerão", "passa pro sincerão", "quero o parecer do sincerão"
- "sincerão, analisa isso", "manda pro sincerão", "o que o sincerão acha?"
- "sincerão" (sozinho, como comando)
- Ou qualquer menção explícita ao nome "sincerão" pedindo análise

Quando invocado manualmente:
1. Colete todo o contexto relevante da conversa (código, plano, implementação, decisão)
2. Se o pedido for sobre **código já implementado**: leia os arquivos relevantes e passe o código + contexto ao Sincerão para análise
3. Se o pedido for sobre **um plano**: passe a descrição completa do plano com objetivos, abordagem e trade-offs
4. Se o pedido for **genérico** ("analisa isso"): use o contexto mais recente da conversa
5. Execute o mesmo loop iterativo (até 7 rodadas) se houver ajustes a fazer
6. Apresente o relatório completo do parecer ao usuário
