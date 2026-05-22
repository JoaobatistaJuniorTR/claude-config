---
name: project-echo-evolution-ideas
description: "Lista de ideias priorizadas para evolução do Echo, discutidas em 2026-05-15"
metadata: 
  node_type: memory
  type: project
  originSessionId: 493160c0-dc9c-4091-bc64-e1e5b663b68c
---

Ideias de evolução discutidas em 2026-05-15, priorizadas por impacto na experiência:

**Alta prioridade (mudam a experiência):**
- Pós-processamento com LLM — corrigir gramática, pontuar, formatar antes de injetar
- Comandos de voz — "ponto final", "nova linha", "apaga tudo" como ações
- Dicionário customizado — termos técnicos que o Whisper erra

**Média prioridade (qualidade de vida):**
- Feedback sonoro — beep ao iniciar/parar gravação
- Clipboard mode — copiar transcrição em vez de injetar
- Histórico navegável — ouvir/retranscrever gravações anteriores
- Seleção de microfone via tray

**Polimento/Distribuição:**
- Ícone próprio (.ico) para exe e tray
- Compilar instalador Inno Setup (script pronto em `build/echo-installer.iss`)
- Auto-update
- Indicador visual de nível de áudio na overlay
- Fallback de modelo STT

**Why:** O usuário quer evoluir incrementalmente. Priorizar features que mudam o dia a dia antes de polimento visual.

**How to apply:** Ao planejar próximas sessões, começar pelas features de alta prioridade. Cada feature deve ser um ciclo independente (spec → plan → implement).
