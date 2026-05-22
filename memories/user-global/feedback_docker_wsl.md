---
name: Docker via WSL
description: Docker deve ser executado via WSL (wsl docker ...), não diretamente no Windows
type: feedback
originSessionId: dd6be78c-3fee-46b4-8b04-d40e4a724956
---
Docker nesta máquina roda dentro do WSL (Dockertron/portable). Sempre usar `wsl docker ...` para comandos Docker.

**Why:** Docker não está no PATH do Windows (nem bash nem PowerShell). O executável está no WSL.

**How to apply:** Todo comando Docker deve ser prefixado com `wsl`. Ex: `wsl docker ps`, `wsl docker compose up`. Isso vale pro slon e qualquer outro contexto.
