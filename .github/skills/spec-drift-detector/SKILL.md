---
name: spec-drift-detector
description: >
  Detecta divergências entre a implementação atual e as specs documentadas nos flows.
  Classifica drifts em bloqueantes (código sem spec), recomendações (spec desatualizada)
  e informativos. Invocada pelo code-reviewer durante revisão pre-PR.
allowed-tools: shell
---

# Skill: Spec Drift Detector

## Objetivo
Detectar divergências entre a implementação atual e as specs documentadas nos flows.
Invocado pelo agente `code-reviewer` durante revisão pre-PR.

## Inputs
- Diff do PR (arquivos modificados)
- Flows do serviço em `flows/services/{service}/flows/`
- OpenAPI do serviço (se disponível)

## Processo de detecção

### 1. Identificar escopo do PR
A partir dos arquivos modificados no diff, identifique:
- Quais Use Cases foram criados/modificados?
- Quais adapters (REST, Kafka, persistence) foram criados/modificados?
- Quais entidades de domínio foram alteradas?

### 2. Mapear para flows
Para cada componente modificado:
- Qual flow em `flows/services/{service}/flows/` corresponde?
- Se não existe flow correspondente → **drift: código sem spec** (bloqueante)

### 3. Comparar comportamento

#### Para Use Cases
- O flow documenta o mesmo happy path implementado?
- Todos os error paths do código estão no flow?
- Novos exceções de domínio estão documentadas?

#### Para REST Adapters
- Todos os endpoints implementados estão no OpenAPI?
- Os status codes de resposta batem?
- Novos parâmetros/campos estão documentados?

#### Para Kafka
- Todos os topics consumidos/publicados estão no flow?
- O schema do payload evolui sem atualização de spec?

### 4. Classificação de drifts

| Drift | Severidade | Ação |
|-------|-----------|------|
| Código sem nenhuma spec | 🔴 Bloqueante | Criar spec antes do merge |
| Comportamento mudou sem atualizar flow | 🟡 Recomendação | Atualizar flow no PR |
| Spec mais detalhada que código | 🔵 Informativo | Registrar como débito técnico |
| Novo error path sem documentação | 🟡 Recomendação | Adicionar ao flow |

## Output
Relatório estruturado entregue ao `code-reviewer`:

```
## Relatório de Drift — {service} — PR #{n}

### 🔴 Bloqueantes
- {componente}: sem spec correspondente em flows/

### 🟡 Recomendações de atualização
- {flow}.md: {o que precisa ser atualizado}

### 🔵 Informativos
- {observação}

### ✅ Sem drift detectado
- {componentes verificados e OK}
```
