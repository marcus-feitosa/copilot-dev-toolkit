---
name: validate-spec
description: >
  Detecta divergências entre a spec documentada (flows .md, OpenAPI) e o código
  implementado. Classifica drifts em bloqueantes, recomendações e informativos.
  Invocada pelo code-reviewer durante revisão pre-PR ou ao suspeitar que o código
  evoluiu sem atualização de spec.
allowed-tools: shell
---

# Skill: Validate Spec vs Implementation

## Objetivo
Detectar divergências entre a spec documentada (flows .md, OpenAPI) e o código implementado.

## Quando usar
- Durante revisão pre-PR (invocado pelo code-reviewer)
- Ao suspeitar que o código evoluiu sem atualização de spec

## Processo de validação

### 1. Mapeamento spec → código
Para cada flow documentado em `flows/services/{service}/flows/`:
- Identifique o use case correspondente (`application/usecase/`)
- Identifique o adapter de entrada (`infrastructure/in/`)
- Identifique os adapters de saída envolvidos (`infrastructure/out/`)

### 2. Checklist de drift

#### Contratos REST
- [ ] Endpoints documentados no OpenAPI existem no Resource?
- [ ] Parâmetros e schemas batem?
- [ ] Códigos de resposta de erro estão implementados?
- [ ] Novos endpoints existem sem spec?

#### Fluxos de negócio
- [ ] Happy path implementado bate com o flow documentado?
- [ ] Error paths documentados estão tratados?
- [ ] Novos error paths foram introduzidos sem documentação?

#### Eventos Kafka
- [ ] Topics documentados batem com o código?
- [ ] Payload schema evoluiu sem atualização de spec?
- [ ] Novos consumers/producers sem documentação?

### 3. Classificação dos drifts
| Tipo | Classificação |
|------|---------------|
| Código sem spec | Bloqueante |
| Spec desatualizada (código mudou) | Recomendação |
| Spec mais completa que o código | Informativo |

## Output
Relatório de drift estruturado entregue ao `code-reviewer` para apresentação HITL.
