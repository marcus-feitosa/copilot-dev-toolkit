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

Os agentes e skills são carregados automaticamente pelo Copilot CLI a partir de `.github/agents/`
e `.github/skills/`. Use `/skills list` para verificar as skills disponíveis e `/agent` para
selecionar um agente interativamente.

---

## Agentes disponíveis

Agentes em `.github/agents/` — invoque com `/agent` ou `--agent {nome}`:

| Agente | Propósito | Quando usar |
|--------|-----------|-------------|
| `domain-extractor` | Gera domain-overview.md + OpenAPI a partir do código | Ao incorporar um serviço existente ao toolkit |
| `spec-writer` | Gera ADRs, OpenAPI, flows | Antes de implementar qualquer feature |
| `harness-runner` | Scaffolding, código, migrations | Após spec aprovada |
| `robot-test-generator` | Testes Robot Framework | Após implementação |
| `code-reviewer` | Qualidade + sync de specs | Antes de abrir PR |
| `hitl-gate` | Checkpoint de aprovação | Invocado automaticamente por outros agentes |

## Skills disponíveis

Skills em `.github/skills/` — invoque com `/{nome}` dentro de prompts:

| Skill | Categoria | Propósito |
|-------|-----------|-----------|
| `/read-service-source` | domain | Acessa repo GitHub ou path local com branch guard |
| `/extract-domain-model` | domain | Guia extração de DDD de código Java/Quarkus |
| `/scaffold-service` | harness | Gera estrutura inicial de microserviço hexagonal |
| `/generate-migration` | harness | Gera scripts Flyway seguros para Aurora PostgreSQL |
| `/write-test-suite` | harness | Gera testes unitários/integração JUnit/Quarkus |
| `/generate-adr` | sdd | Gera Architecture Decision Records |
| `/generate-openapi` | sdd | Gera contratos OpenAPI 3.1 |
| `/validate-spec` | sdd | Detecta drift spec vs implementação |
| `/code-quality` | review | Checklist de qualidade hexagonal Java/Quarkus |
| `/spec-drift-detector` | review | Detecta drifts entre código e flows |
| `/spec-updater` | review | Atualiza flows após aprovação HITL |
| `/robot-conventions` | robot | Convenções e keywords base Robot Framework |
| `/robot-async-patterns` | robot | Padrões para fluxos assíncronos Kafka |
| `/confirm-destructive` | hitl | Checkpoint para ações destrutivas |
| `/summarize-plan` | hitl | Resume plano de execução antes de agir |

```bash
# Exemplos de uso via CLI
copilot --agent domain-extractor --prompt "Extraia o domínio de git@github.com:org/meu-servico.git"
copilot --agent spec-writer --prompt "Gere spec para historia_id=123 flow_id=456"
copilot --agent code-reviewer --prompt "Revise o PR da branch feature/minha-feature"
```

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
.github/
  ├── copilot-instructions.md     # Instruções globais do Copilot (stack, princípios, HITL)
  ├── agents/                     # Custom agents — formato oficial .agent.md
  │   ├── domain-extractor.agent.md
  │   ├── spec-writer.agent.md
  │   ├── harness-runner.agent.md
  │   ├── robot-test-generator.agent.md
  │   ├── code-reviewer.agent.md
  │   └── hitl-gate.agent.md
  └── skills/                     # Skills atômicas — formato oficial SKILL.md
      ├── read-service-source/SKILL.md
      ├── extract-domain-model/SKILL.md
      ├── scaffold-service/SKILL.md
      ├── generate-migration/SKILL.md
      ├── write-test-suite/SKILL.md
      ├── generate-adr/SKILL.md
      ├── generate-openapi/SKILL.md
      ├── validate-spec/SKILL.md
      ├── code-quality/SKILL.md
      ├── spec-drift-detector/SKILL.md
      ├── spec-updater/SKILL.md
      ├── robot-conventions/SKILL.md
      ├── robot-async-patterns/SKILL.md
      ├── confirm-destructive/SKILL.md
      └── summarize-plan/SKILL.md
hooks/                            # Hooks do Copilot CLI (pre-push)
prompts/                          # Prompts reutilizáveis por domínio
instructions/                     # Custom instructions por stack/squad
flows/                            # Fonte de verdade dos fluxos do domínio
  └── services/
      ├── _template/              # Templates de flow e ADR
      └── {service}/              # Flows por serviço
workspace/
  └── domain-map.yml              # Registro de serviços e bounded contexts
docs/                             # Documentação do toolkit
```

---

## Contribuindo

Veja `docs/CONTRIBUTING.md` para adicionar novos agentes, skills ou flows.
