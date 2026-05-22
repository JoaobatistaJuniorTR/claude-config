---
name: project-echo-overview
description: "Echo é um utilitário Push-to-Talk de transcrição de voz para Windows, usando Whisper via LiteLLM proxy corporativo"
metadata: 
  node_type: memory
  type: project
  originSessionId: 493160c0-dc9c-4091-bc64-e1e5b663b68c
---

Echo é um app desktop Windows de Push-to-Talk que grava áudio e transcreve usando Whisper via proxy LiteLLM corporativo (Thomson Reuters).

**Arquitetura:**
- Python + Tkinter (overlay transparente) + pystray (tray icon) + keyboard (hotkeys)
- Thread principal = Tkinter mainloop; threads separadas para tray, hotkey hook e event loop
- `queue.Queue` central para comunicação inter-thread (padrão event-driven)
- ThreadPoolExecutor para chamadas HTTP de transcrição
- Injeção de texto via clipboard backup/restore (não pyautogui — quebra com Unicode/acentos)

**Stack:** Python 3.13, keyboard, sounddevice, scipy, litellm, pystray, Pillow, python-dotenv, PyInstaller (onedir)

**Config:** env vars > ~/.echo/config.json > defaults hardcoded. Secrets ficam só no .env.

**Modelo STT:** `openai/whisper-1` via `https://litellm.int.thomsonreuters.com`. Modelo `groq/whisper-large-v3` não existe no proxy.

**SSL corporativo:** Requer `REQUESTS_CA_BUNDLE` e `SSL_CERT_FILE` apontando para `C:\ProgramData\SSL\certs\os-ca-bundle.pem` no .env.

**Build:** PyInstaller `--onedir` com `collect_all` para litellm, tiktoken e tiktoken_ext. O `--onefile` causava problemas de arquivos não encontrados e lentidão na abertura. Saída em `dist/echo/`.

**Why:** Ferramenta pessoal do usuário para transcrição rápida de voz em qualquer campo de texto no Windows.

**How to apply:** Ao evoluir o Echo, respeitar a arquitetura event-driven com queue central, manter compatibilidade com o proxy corporativo, e sempre rebuildar com PyInstaller onedir após mudanças.
