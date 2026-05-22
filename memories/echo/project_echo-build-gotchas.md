---
name: project-echo-build-gotchas
description: Problemas encontrados e soluções no build PyInstaller do Echo
metadata: 
  node_type: memory
  type: project
  originSessionId: 493160c0-dc9c-4091-bc64-e1e5b663b68c
---

Armadilhas resolvidas durante o build PyInstaller do Echo (2026-05-15):

1. **litellm model_prices JSON não encontrado** → Resolvido com `collect_all('litellm')` no spec
2. **litellm.litellm_core_utils.tokenizers não encontrado** → Mesmo fix, `collect_all` pega tudo
3. **tiktoken encoding cl100k_base não encontrado** → `collect_all('tiktoken')` + `collect_all('tiktoken_ext')` no spec
4. **SSL X509 NO_CERTIFICATE_OR_CRL_FOUND** → `.env` não era encontrado pelo exe (CWD errado). Fix: `echo.py` usa `sys.executable` path quando `sys.frozen=True`. Também requer `SSL_CERT_FILE` e `REQUESTS_CA_BUNDLE` no .env apontando pro cert corporativo.
5. **--onefile lento e problemático** → Trocado para `--onedir`. Descompactação de ~123MB causava delay de vários segundos e erros de arquivo não encontrado. Onedir abre instantâneo.
6. **echo.ico não existe** → `icon=None` no spec até criar o ícone
7. **PermissionError no rebuild** → Fechar o echo.exe antes de rebuildar (PyInstaller precisa deletar dist/echo/)

**Why:** litellm é uma dependência pesada com muitos submódulos, data files e imports dinâmicos que o PyInstaller não detecta automaticamente.

**How to apply:** Ao rebuildar, sempre usar `collect_all` para litellm/tiktoken. Se adicionar nova dependência pesada, considerar `collect_all` preventivamente.
