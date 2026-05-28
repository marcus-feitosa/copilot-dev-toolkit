---
name: generate-adr
description: >
  Gera um Architecture Decision Record (ADR) estruturado a partir de uma decisão técnica.
  Verifica ADRs existentes para evitar duplicidade e salva em flows/services/{service}/adr/
  após aprovação HITL. Use antes de adotar novos padrões arquiteturais ou dependências.
allowed-tools: shell
---

# Skill: Generate ADR

## Objetivo
Gerar um Architecture Decision Record (ADR) estruturado a partir de uma decisão técnica descrita em linguagem natural.

## Quando usar
- Antes de introduzir um novo padrão arquitetural
- Antes de adotar uma nova dependência relevante
- Ao escolher entre abordagens técnicas com trade-offs significativos
- Ao definir um novo contrato entre serviços

## Inputs esperados
- Descrição da decisão a ser tomada
- Contexto do serviço (nome, bounded context)
- Alternativas consideradas (se houver)

## Processo
1. Identifique o bounded context no `workspace/domain-map.yml`
2. Verifique ADRs existentes em `flows/services/{service}/` para evitar conflito ou duplicidade
3. Gere o ADR seguindo o template abaixo
4. Salve em `flows/services/{service}/adr/ADR-{n}-{titulo-kebab}.md`

## Template ADR

```markdown
# ADR-{n}: {Título}

**Status**: Proposto | Aceito | Substituído por ADR-{n}
**Data**: {YYYY-MM-DD}
**Serviço**: {nome do serviço}
**Bounded Context**: {nome do bounded context}

## Contexto
{Descreva o problema ou necessidade que motivou esta decisão}

## Decisão
{Descreva claramente a decisão tomada}

## Alternativas consideradas
| Alternativa | Prós | Contras |
|-------------|------|---------|
| {opção A}   |      |         |
| {opção B}   |      |         |

## Consequências
### Positivas
- {consequência}

### Negativas / Trade-offs
- {trade-off}

## Referências
- {links, PRs, issues relevantes}
```

## Output
Arquivo `.md` no path correto, aguardando aprovação HITL antes de salvar.
