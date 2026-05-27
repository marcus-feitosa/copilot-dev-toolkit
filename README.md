# copilot-dev-toolkit

Repositório central de agentes, skills e prompts para GitHub Copilot CLI.
Suporta o fluxo de desenvolvimento baseado em **SDD**, **Harness Engineering** e **HITL**
para arquiteturas distribuídas Java/Quarkus com arquitetura hexagonal.

---

## Pré-requisitos

- GitHub Copilot CLI instalado (`gh extension install github/gh-copilot`)
- Autenticação GitHub configurada (`gh auth login`)
- Clone este repositório localmente:

```bash
git clone git@github.com:{org}/copilot-dev-toolkit.git
cd copilot-dev-toolkit
```

---

## Agentes disponíveis

| Agente | Propósito | Quando usar |
|--------|-----------|-------------|
| `domain-extractor` | Gera domain-overview.md + OpenAPI a partir do código | Ao incorporar um serviço existente ao toolkit |
| `spec-writer` | Gera ADRs, OpenAPI, flows | Antes de implementar qualquer feature |
| `harness-runner` | Scaffolding, código, migrations | Após spec aprovada |
| `robot-test-generator` | Testes Robot Framework | Após implementação |
| `code-reviewer` | Qualidade + sync de specs | Antes de abrir PR |
| `hitl-gate` | Checkpoint de aprovação | Invocado automaticamente por outros agentes |

---

## Fluxo SDD recomendado

```
0. domain-extractor → (apenas para serviços existentes) gera domain-overview + OpenAPI
1. spec-writer      → spec aprovada via HITL
2. harness-runner   → código + testes unitários
3. robot-test-generator → testes E2E/integração
4. code-reviewer    → revisão + sync de flows
5. PR aberto
```

---

## Configuração por serviço

Adicione `.copilot-toolkit.yml` na raiz de cada repo de serviço:

```yaml
service: {service-name}
bounded-context: {bounded-context}
toolkit:
  flows: flows/services/{service-name}/
  instructions: instructions/java-quarkus.md
```

Registre o serviço em `workspace/domain-map.yml`.

---

## Estrutura do repositório

```
.github/                    # Instruções globais do Copilot
agents/                     # Custom agents (spec-writer, harness-runner, etc.)
skills/                     # Skills atômicas reutilizáveis pelos agentes
  ├── domain/               # Extração de modelo de domínio (domain-extractor)
  ├── sdd/                  # Geração e validação de specs
  ├── harness/              # Scaffolding e geração de código
  ├── review/               # Qualidade e drift detection
  ├── robot/                # Convenções Robot Framework
  └── hitl/                 # Checkpoints de aprovação
hooks/                      # Hooks do Copilot CLI (pre-push)
prompts/                    # Prompts reutilizáveis por domínio
instructions/               # Custom instructions por stack/squad
flows/                      # Fonte de verdade dos fluxos do domínio
  └── services/
      ├── _template/        # Templates de flow e ADR
      └── {service}/        # Flows por serviço
workspace/
  └── domain-map.yml        # Registro de serviços e bounded contexts
docs/                       # Documentação do toolkit
```

---

## Contribuindo

Veja `docs/CONTRIBUTING.md` para adicionar novos agentes, skills ou flows.
