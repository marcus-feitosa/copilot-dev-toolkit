# Contribuindo com o toolkit

## Adicionando um novo serviço ao domínio

1. Registre o serviço em `workspace/domain-map.yml`
2. Crie o diretório `flows/services/{service-name}/`
3. Crie `flows/services/{service-name}/overview.md` descrevendo o serviço
4. Crie `flows/services/{service-name}/flows/` para os flows do serviço
5. Adicione `.copilot-toolkit.yml` na raiz do repo do serviço

## Adicionando um novo agente

1. Crie o diretório `agents/{nome-do-agente}/`
2. Crie `agents/{nome-do-agente}/agent.yml` seguindo a estrutura dos agentes existentes
3. Crie `agents/{nome-do-agente}/README.md` documentando propósito e uso
4. Referencie as skills necessárias no `agent.yml`

## Adicionando uma nova skill

1. Identifique a categoria correta: `sdd/`, `harness/`, `review/`, `robot/`, `hitl/`
2. Crie o arquivo `.md` seguindo o padrão: Objetivo, Inputs, Processo, Output
3. Referencie a skill nos agentes relevantes

## Atualizando um flow existente

Nunca atualize um flow diretamente — use o agente `code-reviewer` que passa pelo HITL.
Se precisar atualizar manualmente:
1. Documente o motivo
2. Adicione entrada no `## Changelog` do flow
3. Abra PR com a atualização referenciando o contexto

## Convenções de nomenclatura

- Agentes: kebab-case (`spec-writer`, `code-reviewer`)
- Skills: kebab-case descritivo (`generate-adr`, `spec-drift-detector`)
- Flows: kebab-case do fluxo (`happy-path`, `error-handling`, `retry-flow`)
- ADRs: `ADR-{n}-{titulo-kebab}.md` (ex: `ADR-001-outbox-pattern.md`)
