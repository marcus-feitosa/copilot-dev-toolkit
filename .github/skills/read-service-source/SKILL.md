---
name: read-service-source
description: >
  Acessa o código-fonte de um microserviço Java/Quarkus a partir de uma referência
  GitHub (SSH, HTTPS ou shorthand) ou de um path local, garantindo que o repositório
  esteja na branch correta (dev ou main) antes de qualquer leitura. Invoque esta skill
  ao iniciar a extração de domínio de qualquer serviço.
allowed-tools: shell
---

# Skill: Read Service Source

## Objetivo

Acessar o código-fonte de um microserviço a partir de uma referência GitHub ou path local,
garantindo que o repositório esteja na branch correta antes de qualquer leitura.

## Detecção do tipo de fonte

| Padrão de entrada | Tipo detectado |
|-------------------|----------------|
| `git@github.com:...` | GitHub SSH |
| `https://github.com/...` | GitHub HTTPS |
| Começa com `/` ou `C:\` | Path local |
| Apenas `{org}/{repo}` | GitHub shorthand |

---

## Fluxo para fonte GitHub

```bash
# 1. Extraia org e repo do input
ORG="{org}"
REPO="{repo}"
DEST="/tmp/domain-extractor/${REPO}"

# 2. Clone raso (sem histórico, sem tags — apenas o código)
gh repo clone "${ORG}/${REPO}" "${DEST}" -- --depth=1 --no-tags --quiet

# 3. Verifique a branch atual
BRANCH=$(git -C "${DEST}" rev-parse --abbrev-ref HEAD)
echo "Branch atual: ${BRANCH}"

# 4. Branch guard
if [ "${BRANCH}" != "dev" ] && [ "${BRANCH}" != "main" ]; then
  echo "⚠️  Branch '${BRANCH}' não é dev nem main. Tentando mudar..."
  git -C "${DEST}" checkout dev 2>/dev/null \
    || git -C "${DEST}" checkout main 2>/dev/null \
    || { echo "❌ Não foi possível mudar para dev ou main."; exit 1; }
  BRANCH=$(git -C "${DEST}" rev-parse --abbrev-ref HEAD)
  echo "✅ Branch alterada para: ${BRANCH}"
fi
```

Após sucesso, `SOURCE_PATH=${DEST}`.

---

## Fluxo para fonte local

```bash
# 1. Valide que o path existe
ls "{path}" || { echo "❌ Path não encontrado: {path}"; exit 1; }

# 2. Verifique que é um repositório git
git -C "{path}" rev-parse --git-dir || { echo "❌ Não é um repositório git."; exit 1; }

# 3. Verifique a branch atual
BRANCH=$(git -C "{path}" rev-parse --abbrev-ref HEAD)
echo "Branch atual: ${BRANCH}"

# 4. Branch guard
if [ "${BRANCH}" != "dev" ] && [ "${BRANCH}" != "main" ]; then
  echo "⚠️  Branch '${BRANCH}' não é dev nem main. Tentando mudar..."
  git -C "{path}" checkout dev 2>/dev/null \
    || git -C "{path}" checkout main 2>/dev/null \
    || { echo "❌ Não foi possível mudar para dev ou main."; exit 1; }
  BRANCH=$(git -C "{path}" rev-parse --abbrev-ref HEAD)
  echo "✅ Branch alterada para: ${BRANCH}"
fi
```

Após sucesso, `SOURCE_PATH={path}`.

---

## Mensagem HITL para branch guard

Quando a branch original não for `dev` ou `main`, apresente ao dev:

```
⚠️  Branch guard acionado

Repositório : {repo}
Branch atual: {branch_original}
Ação        : Tentando checkout automático para 'dev' → 'main'

Se o checkout for bem-sucedido, o processamento continua.
Se falhar, aguardarei sua instrução.
```

---

## Limpeza após uso (somente para fontes GitHub)

```bash
rm -rf /tmp/domain-extractor/{repo}
```

Execute somente após a geração dos arquivos de domínio ter sido concluída com sucesso.

---

## Erros conhecidos e o que fazer

| Erro | Causa provável | Ação |
|------|---------------|------|
| `gh: command not found` | gh CLI não instalado | Informe o dev e pare |
| `Repository not found` | Repo inexistente ou sem permissão | Informe o dev e pare |
| `fatal: not a git repository` | Path local não é um repo git | Informe o dev e pare |
| `error: pathspec 'dev' did not match` | Branch dev não existe | Tente main; se falhar, peça instrução |
| `Your local changes would be overwritten` | Repo local com mudanças não commitadas | Informe o dev — não force o checkout |
