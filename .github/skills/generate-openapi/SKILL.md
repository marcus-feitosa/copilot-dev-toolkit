---
name: generate-openapi
description: >
  Gera contrato OpenAPI 3.1 para endpoints REST de serviços Quarkus/RESTEasy Reactive.
  Inclui convenções de path, response patterns (201/422/409), schemas padrão de erro
  e mapeamento Java→JSON Schema. Invocada pelo spec-writer antes de qualquer implementação.
---

# Skill: Generate OpenAPI Spec

## Objetivo
Gerar contrato OpenAPI 3.1 para endpoints REST de serviços Quarkus/RESTEasy Reactive.

## Quando usar
- Antes de implementar um novo endpoint
- Ao documentar endpoints existentes sem spec
- Ao definir contrato de integração entre serviços

## Inputs esperados
- Descrição do recurso e operações
- Flow documentado ou use case de referência
- Nome do serviço e bounded context

## Processo
1. Leia o flow correspondente em `flows/services/{service}/flows/`
2. Identifique recursos, operações, inputs e outputs
3. Gere o contrato OpenAPI 3.1 completo
4. Inclua exemplos realistas nos schemas
5. Apresente para aprovação HITL antes de salvar

## Convenções para Quarkus/RESTEasy

### Path conventions
- Recursos no plural: `/duplicatas`, `/registros`
- Versão no path: `/v1/duplicatas`
- Sub-recursos: `/v1/duplicatas/{id}/escrituracoes`

### Response patterns
```yaml
# Sucesso com recurso criado
201 Created:
  headers:
    Location: { schema: { type: string, format: uri } }
  content:
    application/json:
      schema: { $ref: '#/components/schemas/{Recurso}Response' }

# Erro de validação (Bean Validation / domínio)
422 Unprocessable Entity:
  content:
    application/json:
      schema: { $ref: '#/components/schemas/ValidationErrorResponse' }

# Erro de negócio
409 Conflict:
  content:
    application/json:
      schema: { $ref: '#/components/schemas/BusinessErrorResponse' }
```

### Schemas padrão de erro
```yaml
ValidationErrorResponse:
  type: object
  properties:
    errors:
      type: array
      items:
        type: object
        properties:
          field: { type: string }
          message: { type: string }

BusinessErrorResponse:
  type: object
  properties:
    code: { type: string }
    message: { type: string }
    correlationId: { type: string }
```

## Output
Arquivo `openapi.yml` em `flows/services/{service}/`, aguardando aprovação HITL.
