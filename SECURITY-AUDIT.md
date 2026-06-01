# Auditoria de Segurança — fork pinado (ciro-rosa)

**Data:** 2026-06-01
**Objetivo:** distribuir com segurança pros alunos, sem que pushes futuros de terceiros
entrem sem revisão.

## O que foi auditado

| Componente | Commit auditado |
|---|---|
| Wrapper MCP (`AndyShaman/gemini-webapi-mcp`) | `de3c5e056144701f4ebb2b68e7c9dc13faf7fba4` |
| Engine (`xob0t/Gemini-API`) | `4402e5231760a1e85e28eb6cd0040ff302b764de` (tag `audited-4402e52` no fork) |

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
- Engine travada no SHA imutável `4402e52` do **fork `ciro-rosa/Gemini-API`** (não no upstream).
- Dependências PyPI diretas travadas em versão exata (`==`) nas versões auditadas.

Resultado: nenhum push do `AndyShaman` ou do `xob0t`, nem release novo das libs diretas,
alcança a instalação dos alunos sem você atualizar este fork e o `.mcpb` conscientemente.

## Como re-auditar antes de subir versão nova
1. `git fetch upstream && git diff <commit-atual>..upstream/main`
2. Revise o diff. Se ok, bump dos SHAs/versões aqui e gere um `.mcpb` novo.
