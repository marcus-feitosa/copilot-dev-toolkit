# copilot-dev-toolkit

Repositório central de agentes, skills e prompts para GitHub Copilot CLI.
Suporta o fluxo de desenvolvimento baseado em **SDD (Specification-Driven Development)**, **Harness Engineering** e **HITL (Human in the Loop)**
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

## Fluxo SDD recomendado

```
[Serviço existente]
  0. domain-extractor → documenta domínio no Confluence (domain-overview + OpenAPI)

[Feature nova]
  1. spec-writer      → gera spec (ADR / OpenAPI / flow) aprovada via HITL
  2. harness-runner   → scaffolding + código + migrations (requer spec aprovada)
  3. robot-test-generator → suítes Robot Framework E2E/integração
  4. code-reviewer    → revisão de qualidade + sync de flows
  5. PR aberto
```

> Nunca use `harness-runner` sem spec aprovada. Nunca abra PR sem passar pelo `code-reviewer`.

---

## Como usar cada agente

### `domain-extractor` — Extrair domínio de serviço existente

**Quando usar:** ao incorporar um serviço já existente ao toolkit — antes de qualquer desenvolvimento.

**O que o agente precisa:**
- Referência do repositório: URL SSH/HTTPS do GitHub, shorthand `org/repo` ou caminho local
- Nome do domínio (será o título da página raiz no Confluence)
- O repositório deve estar na branch `dev` ou `main`

**Prompts ideais:**

```
# Um serviço via GitHub SSH
@domain-extractor Extraia o domínio do serviço git@github.com:org/registro-duplicatas.git para o domínio "Registro"

# Shorthand GitHub
@domain-extractor Extraia o domínio de org/escrituracao-titulos para o domínio "Escrituração"

# Múltiplos serviços do mesmo domínio (acumulativo)
@domain-extractor Extraia o domínio dos serviços org/registro-duplicatas e org/cedente-service para o domínio "Cedente"

# Caminho local
@domain-extractor Extraia o domínio de C:\projetos\meu-servico para o domínio "Pagamento"
```

**O que informar para melhores resultados:**
- Informe o nome do domínio exatamente como ele deve aparecer no Confluence
- Se houver múltiplos repositórios, liste todos de uma vez para gerar o flow E2E consolidado
- Se a branch não for `dev`/`main`, informe isso e aguarde a instrução do agente

**O que o agente vai fazer (com HITL):**
1. Clona o repositório e verifica branch
2. Lê o código Java/Quarkus e extrai modelo de domínio, ports, use cases, Kafka, REST
3. Apresenta plano de execução para aprovação (`sim / não / ajustar`)
4. Solicita aprovação **por página** antes de criar/atualizar no Confluence
5. Gera: página de domínio, página OpenAPI 3.1, flow E2E cross-service

**Checkpoints HITL:** aprovação do plano + aprovação por página do Confluence

---

### `spec-writer` — Gerar especificação antes de implementar

**Quando usar:** antes de iniciar qualquer implementação de feature ou mudança de contrato.

**O que o agente precisa:**
- Nome do domínio (deve existir no Confluence — gerado pelo `domain-extractor`)
- `historia_id`: ID da página do Confluence ou chave Jira da história de negócio
- `flow_id`: ID do flow documentado que será detalhado/derivado

**Prompts ideais:**

```
# Gerar OpenAPI para nova operação
@spec-writer Gere spec OpenAPI para o domínio "Registro", historia_id=CONF-789, flow_id=FLOW-001

# Gerar ADR para decisão arquitetural
@spec-writer Gere ADR para o domínio "Escrituração", serviço escrituracao-titulos — decisão: usar Outbox Pattern para publicação Kafka. historia_id=CONF-812, flow_id=FLOW-003

# Gerar flow de novo caso de uso
@spec-writer Documente o flow de cancelamento de duplicata no domínio "Registro", historia_id=CONF-901, flow_id=FLOW-002
```

**O que informar para melhores resultados:**
- Sempre forneça `historia_id` e `flow_id` — sem eles o agente não tem rastreabilidade
- Se ainda não existe flow para a história, informe isso: o agente criará um novo flow derivado
- Descreva brevemente o que a feature deve fazer — o agente usa isso junto com a história para gerar a spec

**O que o agente vai fazer (com HITL):**
1. Consulta o domínio no Confluence via `lookup-domain-confluence`
2. Lê a história de negócio e o flow de referência
3. Avalia ambiguidades — pergunta ao dev antes de prosseguir se algum requisito não estiver claro
4. Solicita permissão explícita antes de ler qualquer arquivo local (templates, código-fonte)
5. Gera spec (ADR, OpenAPI 3.1 ou flow .md conforme contexto) com checklist de completude para `harness-runner`
6. Refina iterativamente com o dev até todos os itens do checklist estarem ✅
7. Salva localmente em `flows/services/{service}/` e publica no Confluence

**Checkpoints HITL:** esclarecimento de ambiguidades → permissão de leitura de arquivos locais → refinamento iterativo da spec → aprovação final antes de qualquer escrita

---

### `harness-runner` — Gerar código a partir de spec aprovada

**Quando usar:** após a spec estar aprovada no Confluence — nunca antes.

**O que o agente precisa:**
- Nome do domínio
- Nome do serviço (o agente lista os disponíveis no Confluence para você escolher)
- A spec OpenAPI deve existir no Confluence e **não pode estar em status draft**

**Prompts ideais:**

```
# Gerar scaffolding completo de serviço novo
@harness-runner Gere o serviço "registro-duplicatas" para o domínio "Registro" a partir da spec aprovada

# Gerar apenas use cases e testes
@harness-runner Gere os use cases e testes para o domínio "Registro", serviço "registro-duplicatas", feature: cancelamento de duplicata

# Gerar migration de banco
@harness-runner Gere a migration Flyway para o domínio "Registro", serviço "registro-duplicatas" — adicionar coluna status_cancelamento
```

**O que informar para melhores resultados:**
- Informe se é scaffolding completo (serviço novo) ou apenas partes específicas (use cases, adapters, migrations)
- Para migrations, descreva a mudança de schema esperada
- Se houver topologia Kafka específica (nomes de topics, consumer groups), mencione

**O que o agente vai fazer (com HITL):**
1. Consulta domínio e lê spec aprovada no Confluence
2. Valida que a spec não está em draft
3. Apresenta plano detalhado de arquivos a criar/modificar
4. Aguarda aprovação antes de escrever qualquer arquivo
5. Gera código Java/Quarkus hexagonal, migrations Flyway e testes JUnit

**Checkpoints HITL:** aprovação do plano + aprovação antes de cada `write_file` e `run_terminal_command`

> **Atenção:** se a spec estiver em draft o agente vai parar e informar — aprove a spec no Confluence antes de prosseguir.

---

### `robot-test-generator` — Gerar suítes de testes Robot Framework

**Quando usar:** após a implementação estar completa, antes de abrir PR.

**O que o agente precisa:**
- Nome do domínio
- Nome do serviço
- Flows e OpenAPI devem estar documentados no Confluence

**Prompts ideais:**

```
# Gerar todos os testes do serviço
@robot-test-generator Gere as suítes Robot para o domínio "Registro", serviço "registro-duplicatas"

# Gerar testes de um flow específico
@robot-test-generator Gere testes Robot para o flow de cancelamento de duplicata no domínio "Registro", serviço "registro-duplicatas"

# Gerar apenas testes de error paths
@robot-test-generator Gere testes de error path Robot para o domínio "Escrituração", serviço "escrituracao-titulos"
```

**O que informar para melhores resultados:**
- Informe se quer cobertura completa (happy path + error paths) ou foco específico
- Se houver dependências externas (outros serviços, mocks necessários), mencione
- Informe o ambiente alvo se houver variações (dev, staging)

**O que o agente vai fazer (com HITL):**
1. Lê flows e OpenAPI do Confluence
2. Identifica dependências (Kafka topics, serviços externos)
3. Apresenta plano de cenários de teste para aprovação
4. Gera suítes `.robot` com keywords reutilizáveis, variáveis por ambiente e padrões async Kafka
5. Cria `tests/robot/README.md` com instruções de execução

**Checkpoints HITL:** aprovação do plano de cenários antes de gerar os arquivos

---

### `code-reviewer` — Revisar qualidade e sync de specs antes do PR

**Quando usar:** antes de abrir qualquer Pull Request.

**O que o agente precisa:**
- Nome do domínio
- Código já na working tree ou em uma branch específica
- Specs documentadas no Confluence para comparar

**Prompts ideais:**

```
# Revisão completa antes de PR
@code-reviewer Revise o domínio "Registro", serviço "registro-duplicatas" na branch feature/cancelamento

# Revisão focada em drift de spec
@code-reviewer Verifique se o código do domínio "Escrituração" está em sync com as specs do Confluence

# Revisão de qualidade apenas
@code-reviewer Revise qualidade hexagonal do serviço "registro-duplicatas" no domínio "Registro"
```

**O que informar para melhores resultados:**
- Informe a branch ou confirme que o código atual é o que deve ser revisado
- Se houver partes que você sabe que mudaram sem atualização de spec, mencione — agiliza a detecção de drift
- Se quiser revisão parcial (só qualidade, só drift), informe o foco

**O que o agente vai fazer (com HITL):**
1. Lê specs e flows do Confluence como referência
2. Analisa qualidade: violações hexagonais, observabilidade (MDC), SRP, cobertura de testes
3. Detecta drift: código sem spec, comportamentos mudados sem documentação
4. Apresenta relatório classificado: **bloqueante** (impede merge) e **recomendação**
5. Para drifts aprovados, propõe atualização de flows no Confluence

**Checkpoints HITL:** aprovação do relatório + aprovação de cada atualização de flow no Confluence

> **Problemas bloqueantes** impedem aprovação do PR até resolução. O agente lista exatamente o que precisa ser corrigido.

---

### `hitl-gate` — Checkpoint explícito de aprovação

**Quando usar:** invocado automaticamente pelos outros agentes em pontos críticos. Pode ser chamado diretamente quando você quer apresentar um plano antes de agir.

**Prompts ideais:**

```
# Apresentar plano antes de executar ação crítica
@hitl-gate Apresente o plano para deletar os arquivos de configuração antigos do serviço "registro-duplicatas"

# Revisar escopo antes de harness
@hitl-gate Valide o escopo do harness antes de gerar código: domínio "Registro", serviço "registro-duplicatas", feature: cancelamento
```

**Como responder ao HITL-gate:**
- `sim` — aprova e o agente prossegue
- `não` — cancela a ação, o agente informa o motivo ao solicitante
- `ajustar escopo` — o agente reapresenta o plano com as modificações informadas

---

## Entendendo o protocolo HITL

O HITL (Human in the Loop) é um princípio central do toolkit: **nenhum agente escreve, deleta ou executa sem sua aprovação explícita.**

### Tipos de checkpoint

| Nível | Quando aparece | Como aprovar |
|-------|---------------|--------------|
| **Plano** | Antes de qualquer ação | `sim` / `não` / `ajustar escopo` |
| **Destrutivo** | Delete de arquivo, DROP/ALTER em migration, sobrescrita de spec aprovada | digitar `CONFIRMO` em maiúsculas |

### O que exige aprovação

- Criação ou atualização de páginas no Confluence
- Escrita de qualquer arquivo no repositório (`write_file`)
- Execução de comandos no terminal (`run_terminal_command`)
- Migrations com `DROP` ou `ALTER`
- Sobrescrita de flows já aprovados

### O que não exige aprovação

- Leitura de arquivos (`read_file`, `list_directory`)
- Consultas ao Confluence (search, get_page)
- Geração de planos e rascunhos (só visualização)

> **Silêncio não é aprovação.** O agente aguarda sua resposta explícita. Se você quiser cancelar, responda `não`.

---

## Skills disponíveis

Skills em `.github/skills/` — invocadas automaticamente pelos agentes ou diretamente em prompts:

| Skill | Categoria | Propósito |
|-------|-----------|-----------|
| `/read-service-source` | domain | Acessa repo GitHub ou path local com branch guard |
| `/extract-domain-model` | domain | Guia extração de DDD de código Java/Quarkus |
| `/lookup-domain-confluence` | domain | Localiza páginas de domínio no Confluence |
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
| `/confirm-destructive` | hitl | Checkpoint para ações destrutivas (requer `CONFIRMO`) |
| `/summarize-plan` | hitl | Resume plano de execução antes de agir |

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
      ├── lookup-domain-confluence/SKILL.md
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
      ├── _template/              # Templates de flow, domain-overview e ADR
      └── {service}/              # Flows por serviço
workspace/
  └── domain-map.yml              # Registro de serviços e bounded contexts
docs/                             # Documentação do toolkit
  ├── CONTRIBUTING.md
  └── HITL-PROTOCOL.md
```

---

## Contribuindo

Veja `docs/CONTRIBUTING.md` para adicionar novos agentes, skills ou flows.
