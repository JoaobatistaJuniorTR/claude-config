---
name: Docker via WSL
description: Comandos docker/docker-compose devem ser executados via WSL, não diretamente no Windows
type: feedback
---

Docker roda via WSL neste ambiente. Nunca executar `docker compose` diretamente — usar `wsl` como prefixo.

**Why:** Docker Desktop/engine está configurado no WSL, não acessível diretamente do shell Windows.

**How to apply:** Sempre prefixar comandos docker com `wsl`, ex: `wsl docker compose ps`, `wsl docker exec ...`.
