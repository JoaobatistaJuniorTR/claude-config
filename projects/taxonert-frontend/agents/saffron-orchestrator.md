# Saffron Orchestrator

**Este arquivo NAO e um agente para ser spawnado.** E um roteiro que a sessao principal do Claude Code segue diretamente. Quando o usuario referencia `@saffron-orchestrator`, leia este arquivo e execute o pipeline voce mesmo, spawnando cada fase como um Agent separado (cada um cria seu proprio bonequinho visivel no UI).

---

## Deteccao de Cenario

Analise o input do usuario e determine o cenario:

| Input | Cenario | Agentes Envolvidos |
|---|---|---|
| URL Figma (`figma.com/design/...`) ou mencao a Figma | figma | designer → specialist → developer → auditor |
| Imagem, screenshot, mockup (caminho de arquivo ou upload) | image | designer → specialist → developer → auditor |
| Descricao textual ("crie uma tela com...") | prompt | designer → specialist → developer → auditor |
| "Migrar tela X para Saffron" | migration | specialist → developer → auditor |
| Ajuste em tela existente ("troque X", "adicione Y", "mude Z") | adjustment | specialist* → developer → auditor |

*No cenario `adjustment`, o specialist so e acionado se o ajuste envolver novos componentes Saffron. Ajustes simples vao direto ao developer.

---

## Pipeline de Execucao

Voce (sessao principal) coordena o pipeline. Cada fase e um Agent spawnado diretamente — isso garante que cada um aparece como **bonequinho separado** no UI.

```
FASE 1 — DESIGN (Blueprint)
  Spawnar Agent: "saffron-figma-designer" (ou "saffron-visual-designer")
  Receber: Design Blueprint no resultado do Agent
  (sem pausa — seguir direto para fase 2)

FASE 2 — MAPEAMENTO SAFFRON (Saffron Guide)
  Spawnar Agent: "saffron-specialist"
  Passar: Blueprint no prompt
  Receber: Saffron Guide no resultado do Agent

FASE 3 — IMPLEMENTACAO
  Spawnar Agent: "saffron-angular-developer"
  Passar: Blueprint + Saffron Guide no prompt
  Receber: Codigo + lista de arquivos no resultado do Agent
  PAUSAR — apresentar implementacao ao usuario para validacao
  Se usuario pedir ajustes → spawnar developer novamente com feedback

FASE 4 — AUDITORIA (paralela)
  Spawnar EM PARALELO (ambos no mesmo bloco de tool calls):
    - Agent: "saffron-auditor" → Relatorio A1-A10 (Saffron compliance)
    - Agent: "angular-auditor" → Relatorio B1-B6 (qualidade Angular)
  Passar: caminhos dos arquivos no prompt de ambos
  Receber: Dois relatorios independentes
  Apresentar resultado unificado ao usuario
```

A unica pausa e na fase 3 (implementacao) para o usuario validar o codigo. **NUNCA** pular a fase 4 (auditoria). Se o usuario pedir "gera tudo completo" ou "sem pausas", pular a pausa da fase 3 tambem, mas manter a fase 4.

---

## Como Spawnar Cada Fase

Cada agente e spawnado com a tool `Agent` diretamente pela sessao principal. Incluir no prompt:
1. Instrucao para ler o agent file correspondente
2. Todo o contexto necessario (input do usuario, Blueprint, Guide, etc.)
3. Instrucao para retornar o resultado no output final

### Fase 1 — Designer

```
Agent:
  subagent_type: "general-purpose"
  name: "saffron-figma-designer"   (ou "saffron-visual-designer")
  prompt: |
    Leia seu agent file em .claude/agents/saffron-figma-designer.md e siga as instrucoes.

    [Contexto do usuario: URL Figma, imagem, ou descricao textual]
    [Instrucoes adicionais se houver]

    Retorne o Design Blueprint completo como seu output final.
```

### Fase 2 — Specialist

```
Agent:
  subagent_type: "general-purpose"
  name: "saffron-specialist"
  prompt: |
    Leia seu agent file em .claude/agents/saffron-specialist.md e siga as instrucoes.

    Aqui esta o Design Blueprint para mapear:
    [conteudo completo do Blueprint recebido na fase 1]

    Retorne o Saffron Guide completo como seu output final.
```

### Fase 3 — Developer

```
Agent:
  subagent_type: "general-purpose"
  name: "saffron-angular-developer"
  prompt: |
    Leia seu agent file em .claude/agents/saffron-angular-developer.md e siga as instrucoes.

    Design Blueprint:
    [conteudo do Blueprint]

    Saffron Guide:
    [conteudo do Guide]

    [Instrucoes adicionais: feature path, integracao com componente pai, etc.]

    Retorne a lista de arquivos criados/modificados como seu output final.
```

### Fase 4 — Auditoria (PARALELA — spawnar ambos no mesmo bloco de tool calls)

```
Agent 1 (paralelo):
  subagent_type: "general-purpose"
  name: "saffron-auditor"
  prompt: |
    Leia seu agent file em .claude/agents/saffron-auditor.md e siga as instrucoes.

    Arquivos para auditar:
    [lista de caminhos recebida na fase 3]

    Retorne o relatorio A1-A10 completo como seu output final.

Agent 2 (paralelo):
  subagent_type: "general-purpose"
  name: "angular-auditor"
  prompt: |
    Leia seu agent file em .claude/agents/angular-auditor.md e siga as instrucoes.

    Arquivos para auditar:
    [lista de caminhos recebida na fase 3]

    Retorne o relatorio B1-B6 completo como seu output final.
```

---

## Pipeline de Ajuste (cenario: adjustment)

**Todo ajuste em tela existente passa pelo pipeline.** Nao importa se e simples ou complexo — o specialist valida o uso correto de Saffron e o auditor garante conformidade A1-A10.

### Deteccao de Ajuste

Indicadores de que e um ajuste (nao criacao):
- Menciona tela/componente que ja existe ("na tela X", "no componente Y")
- Verbos de modificacao ("troque", "adicione", "remova", "mude", "ajuste", "corrija")
- Referencia a elementos especificos ("o botao de pesquisa", "a coluna Z da tabela")

### Fases do Ajuste (sempre 3 fases, sendo a ultima paralela)

```
Fase 1 — SPECIALIST
  Spawnar "saffron-specialist"
  Passar: descricao do ajuste + caminhos dos arquivos existentes
  O specialist analisa o codigo atual e produz um mini Saffron Guide com:
  - Props nativas dos componentes envolvidos (ex: justify-content no saf-layout-grid)
  - Tokens corretos para valores visuais
  - Slots e eventos relevantes
  - Avisos sobre shadow DOM ou limitacoes

Fase 2 — DEVELOPER
  Spawnar "saffron-angular-developer"
  Passar: descricao do ajuste + Saffron Guide do specialist + caminhos dos arquivos
  O developer implementa seguindo o Guide

Fase 3 — AUDITORIA (paralela)
  Spawnar EM PARALELO (ambos no mesmo bloco de tool calls):
    - "saffron-auditor" → A1-A10 (Saffron compliance)
    - "angular-auditor" → B1-B6 (qualidade Angular)
  Passar: caminhos dos arquivos modificados para ambos
  Ambos auditam e corrigem independentemente
```

### Contexto para o Specialist (ajuste)

```
Agent:
  name: "saffron-specialist"
  prompt: |
    Leia seu agent file em .claude/agents/saffron-specialist.md e siga as instrucoes.

    AJUSTE em tela existente. Analise o codigo atual e produza um mini Saffron Guide
    focado no ajuste solicitado. Verifique props nativas dos componentes envolvidos
    via MCP antes de sugerir solucoes CSS custom.

    Arquivos da tela:
    [caminhos dos arquivos .ts, .html, .scss]

    Ajuste solicitado: [descricao do ajuste]

    Retorne o Saffron Guide focado no ajuste como seu output final.
```

### Contexto para o Developer (ajuste)

```
Agent:
  name: "saffron-angular-developer"
  prompt: |
    Leia seu agent file em .claude/agents/saffron-angular-developer.md e siga as instrucoes.

    AJUSTE em tela existente. Siga o Saffron Guide do specialist.
    NUNCA use classes CSS custom ou divs wrapper para algo que o componente Saffron
    ja resolve com props nativas.

    Arquivos da tela:
    [caminhos dos arquivos .ts, .html, .scss]

    Saffron Guide:
    [conteudo do Guide do specialist]

    Ajuste solicitado: [descricao do ajuste]

    Retorne a lista de arquivos modificados como seu output final.
```

---

## Formato do Blueprint (produzido pelo designer)

```markdown
## Blueprint: [nome-da-tela]

### Layout
- Estrutura: [descricao do grid/layout]
- Breakpoints: [comportamento responsivo]

### Componentes
| # | Elemento | Sugestao saf-* | Props Identificadas |
|---|---|---|---|

### Propriedades Visuais
- Cores: [lista de cores encontradas]
- Espacamentos: [valores de spacing]
- Tipografia: [fontes/tamanhos]

### Dados
- Campos: [campos de dados identificados]
- Acoes: [botoes/links e seus propositos]
- Tabelas: [se houver — colunas, dados, paginacao]

### Notas
- [observacoes especiais, warnings]
```

---

## Formato do Saffron Guide (produzido pelo specialist)

```markdown
## Saffron Guide: [nome-da-tela]

### Mapeamento de Componentes
| Elemento | Componente Saffron | Props | Slots | Eventos |
|---|---|---|---|---|

### Mapeamento de Tokens
| Valor Original | Token Saffron |
|---|---|

### Registros no main.ts
- SafX(), SafY(), ...

### Acessibilidade
| Componente | Atributos ARIA |
|---|---|

### Padroes do Projeto
- [referencias a codigo existente relevante]

### Avisos
- [componentes sem equivalente Saffron, limitacoes]
```

---

## Cenario: Migracao

Para migracoes, o fluxo e simplificado (sem designer):

1. Sessao principal analisa o codigo existente e monta um "Blueprint" da tela atual
2. Spawnar specialist para mapeamento Saffron
3. Spawnar developer com skill @saffron-migration
4. Spawnar auditor

---

## Regras

- **Nunca gerar codigo diretamente** — sempre delegar ao developer via Agent
- **Nunca pular auditoria** — mesmo se o usuario pedir "rapido" ou "completo"
- **Apresentar cada fase** ao usuario de forma clara e resumida
- **Cada fase = um Agent separado** — para visibilidade no UI (bonequinho)
- **Passar contexto completo no prompt** de cada Agent — eles nao tem acesso ao historico da conversa
- **Se qualquer agente falhar**, informar o usuario e sugerir alternativa
- **Se o usuario mudar de ideia** no meio do pipeline, adaptar sem recomecar do zero

---

## Tratamento de Erros

| Situacao | Acao |
|---|---|
| MCP Figma falha | Pedir screenshot → spawnar visual-designer em vez de figma-designer |
| Designer nao entende o input | Pedir mais contexto ao usuario |
| Specialist nao encontra equivalente | Incluir warning no Guide, developer decide |
| Developer encontra problema | Spawnar specialist pontualmente para consulta |
| Auditoria falha apos 3 iteracoes | Reportar falhas restantes ao usuario |
