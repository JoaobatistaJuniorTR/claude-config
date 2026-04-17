---
name: saffron-prompt-to-design
description: Traduz descricoes textuais em designs de tela estruturados para Angular + Saffron. Usado pelo saffron-visual-designer no modo prompt.
---

# Prompt to Design — Saffron

## Proposito

Transformar uma descricao textual do usuario em um design de tela estruturado, mapeando para componentes Saffron e padroes de layout do projeto.

---

## Processo de Interpretacao

### 1. Extrair Requisitos

Da descricao do usuario, identificar:

- **Tipo de tela**: listagem, formulario, dashboard, detalhes, CRUD
- **Dados principais**: entidades, campos, tipos
- **Acoes**: criar, editar, excluir, pesquisar, exportar, importar
- **Fluxo**: passo a passo, tabs, wizard, simples
- **Restricoes**: mencionou componentes especificos, layout, comportamento

### 2. Selecionar Template de Layout

Baseado no tipo de tela, aplicar o template mais adequado:

#### Listagem com Filtros (mais comum)
```
saf-container
  saf-layout-grid spacing="5"
    saf-layout-grid-item xs="12"           → Titulo/header
    saf-layout-grid-item xs="12"
      saf-card                             → Card de filtros
        saf-layout-grid spacing="3"
          saf-layout-grid-item xs="12" md="4"  → Campo 1
          saf-layout-grid-item xs="12" md="4"  → Campo 2
          saf-layout-grid-item xs="12" md="4"  → Campo 3
          saf-layout-grid-item xs="12"
            Botoes: Limpar + Pesquisar
    saf-layout-grid-item xs="12"
      saf-card                             → Resultados
        saf-toolbar                        → Acoes da tabela
        wj-flex-grid                       → Tabela
        saf-pagination                     → Paginacao
```

#### Formulario de Cadastro/Edicao
```
saf-container
  saf-layout-grid spacing="5"
    saf-layout-grid-item xs="12"           → Titulo
    saf-layout-grid-item xs="12"
      saf-card                             → Formulario
        saf-layout-grid spacing="3"
          saf-layout-grid-item xs="12" md="6"  → Campos (2 colunas)
          ...
    saf-layout-grid-item xs="12"
      Botoes: Cancelar + Salvar
```

#### Dashboard com Metricas
```
saf-container
  saf-layout-grid spacing="5"
    saf-layout-grid-item xs="12"           → Titulo + filtros
    saf-layout-grid-item xs="12" md="6" lg="3"  → Card metrica 1
    saf-layout-grid-item xs="12" md="6" lg="3"  → Card metrica 2
    saf-layout-grid-item xs="12" md="6" lg="3"  → Card metrica 3
    saf-layout-grid-item xs="12" md="6" lg="3"  → Card metrica 4
    saf-layout-grid-item xs="12"
      saf-card                             → Tabela/grafico principal
```

#### Detalhes de Registro
```
saf-container
  saf-layout-grid spacing="5"
    saf-layout-grid-item xs="12"           → Header com botao voltar
    saf-layout-grid-item xs="12" md="8"
      saf-card                             → Dados principais (2-3 colunas)
    saf-layout-grid-item xs="12" md="4"
      saf-card                             → Dados laterais/resumo
    saf-layout-grid-item xs="12"
      saf-card                             → Historico/timeline
```

#### Tela com Tabs
```
saf-container
  saf-layout-grid spacing="5"
    saf-layout-grid-item xs="12"           → Titulo + info
    saf-layout-grid-item xs="12"
      saf-tabs
        saf-tab label="Tab 1"             → Conteudo tab 1
        saf-tab label="Tab 2"             → Conteudo tab 2
```

### 3. Mapear Campos para Componentes

| Tipo de Campo | Componente Saffron |
|---|---|
| Texto livre | saf-text-field |
| Busca | saf-search-field |
| Selecao (lista curta) | saf-select + saf-option |
| Data | saf-text-field type="date" |
| Checkbox / toggle | saf-checkbox ou saf-switch |
| Radio | saf-radio-group + saf-radio |
| Botao primario | saf-button appearance="primary" |
| Botao secundario | saf-button appearance="secondary" |
| Botao outline | saf-button appearance="outline" |
| Botao icon-only | saf-button com saf-icon + a11y-aria-label |
| Link | saf-anchor |
| Tabela de dados | Wijmo FlexGrid |

### 4. Mapear Acoes para Patterns

| Acao Descrita | Pattern Saffron |
|---|---|
| "pesquisar/filtrar" | Form fields + saf-button appearance="primary" |
| "limpar filtros" | saf-button appearance="outline" |
| "criar novo" | saf-button appearance="primary" com saf-icon slot="start" |
| "exportar" | saf-button appearance="outline" com icone download |
| "editar linha" | saf-button icon-only na coluna da tabela |
| "excluir" | saf-button icon-only + saf-dialog de confirmacao |
| "ver detalhes" | saf-anchor ou navegacao via router |
| "voltar" | saf-button appearance="outline" com icone arrow-left |
| "paginacao" | saf-pagination (se tabela presente) |

---

## Perguntas de Esclarecimento

Se a descricao for incompleta, perguntar (em ordem de prioridade):

1. **Campos da tabela**: "Quais colunas devem aparecer na tabela?"
2. **Campos do formulario**: "Quais campos o formulario deve ter?"
3. **Acoes**: "Alem de pesquisar, quais acoes o usuario pode fazer?"
4. **Layout**: "Prefere filtros em card separado ou na mesma area da tabela?"
5. **Navegacao**: "Esta tela faz parte de algum fluxo maior (tabs, wizard)?"

Nao perguntar tudo de uma vez — priorizar as 1-2 perguntas mais importantes.

---

## Regras

- **Sempre confirmar** o design proposto com o usuario antes de finalizar
- **Preferir simplicidade** — nao adicionar componentes que o usuario nao pediu
- **Seguir padroes do projeto** — usar os mesmos layouts encontrados em `saldo-credor-posterior` e `conciliacao`
- **Tabela = Wijmo** — sempre que houver tabela de dados, usar FlexGrid
- **Responsivo por padrao** — sempre incluir breakpoints xs e md
- **Nao inventar dados** — se o usuario nao mencionou campos especificos, perguntar
