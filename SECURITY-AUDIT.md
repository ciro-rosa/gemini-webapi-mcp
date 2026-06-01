# Auditoria de Segurança — fork pinado (ciro-rosa)

**Data:** 2026-06-01
**Objetivo:** distribuir com segurança pros alunos, sem que pushes futuros de terceiros
entrem sem revisão.

## O que foi auditado

| Componente | Commit auditado |
|---|---|
| Wrapper MCP (`AndyShaman/gemini-webapi-mcp`) | `de3c5e056144701f4ebb2b68e7c9dc13faf7fba4` |
| Engine (`xob0t/Gemini-API`) | `4402e5231760a1e85e28eb6cd0040ff302b764de` |

## Resultado

Leitura completa de `server.py`, `client.py`, `constants.py`, `get_access_token.py`,
`rotate_1psidts.py`, `upload_file.py`, `load_netscape_cookies.py` + varredura de padrões
perigosos em todo o pacote.

- ❌ Sem `eval`/`exec`/`subprocess` malicioso, `socket`, `pickle`, `base64`, `__import__`.
- ✅ **Todo o tráfego de rede vai só pra domínios do Google** (endpoints centralizados em `constants.py`): `google.com`, `gemini.google.com`, `accounts.google.com`, `push.clients6.google.com`, `*.googleusercontent.com`.
- ✅ **Cookies nunca são logados** nem enviados a terceiros — só pro Google.
- ✅ Sem telemetria, sem backdoor, sem download de código arbitrário (o único download externo, `install_gwt.py`, verifica SHA256 e é opcional).

Conclusão: o código faz o que promete. Risco residual NÃO está no código, e sim em:
fragilidade (Google muda o front), Termos de Uso do Google (acesso por cookie é área
cinzenta — risco de flag da conta), e exposição dos cookies (= sessão Google) de cada usuário.

## Pinagem aplicada

`pyproject.toml` deste fork:
- Engine travada na tag **`v1.19.3`** do **fork `ciro-rosa/Gemini-API`** (não no upstream).
  Essa tag = código-fonte auditado do commit `4402e52` + um patch SÓ de metadata de build
  (versão estática no lugar do `setuptools_scm`, pra o `uv` conseguir buildar quando pinado).
  Nenhuma linha de código `.py` foi alterada em relação ao auditado.
- Dependências PyPI diretas travadas em versão exata (`==`) nas versões auditadas.

Resultado: nenhum push do `AndyShaman` ou do `xob0t`, nem release novo das libs diretas,
alcança a instalação dos alunos sem você atualizar este fork e o `.mcpb` conscientemente.

## Mudanças DESTE fork além do pin (código próprio, não-upstream)

- **`gemini_generate_video`** (tool nova em `server.py`): dispara geração de vídeo (Veo),
  faz polling da conversa via `READ_CHAT` e baixa o `.mp4` quando pronto. Recupera a URL por
  **varredura recursiva** procurando `usercontent.google.com/download` (robusto a mudança de
  layout), em vez de índices fixos. Só fala com endpoints do Google; salva em `~/Videos/gemini`.
  Tag de release: `v1.1-pinned`.

## Como re-auditar antes de subir versão nova
1. `git fetch upstream && git diff <commit-atual>..upstream/main`
2. Revise o diff. Se ok, bump dos SHAs/versões aqui e gere um `.mcpb` novo.
