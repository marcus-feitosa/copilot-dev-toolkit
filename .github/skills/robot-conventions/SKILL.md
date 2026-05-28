---
name: robot-conventions
description: >
  Convenções, estrutura de diretórios e keywords base para testes Robot Framework
  nos serviços Java/Quarkus da plataforma. Define common.resource, kafka_helpers.resource,
  template de suite .robot, variáveis obrigatórias e tags padrão.
allowed-tools: shell
---

# Skill: Robot Framework Conventions

## Objetivo
Referência de convenções, padrões e keywords base para testes Robot Framework
nos serviços Java/Quarkus da plataforma.

## Estrutura de diretórios padrão

```
tests/robot/
├── resources/
│   ├── keywords/
│   │   ├── common.resource
│   │   ├── {service}_api.resource
│   │   └── kafka_helpers.resource
│   └── variables/
│       ├── common.yaml
│       ├── dev.yaml
│       └── staging.yaml
├── suites/
│   └── {service}/
│       ├── {flow}_happy_path.robot
│       └── {flow}_error_cases.robot
└── README.md
```

## common.resource — Keywords compartilhadas

```robot
*** Settings ***
Library    RequestsLibrary
Library    Collections
Library    String

*** Keywords ***
Criar Sessao API
    [Arguments]    ${base_url}    ${service}
    Create Session    ${service}    ${base_url}    verify=True

Validar Schema JSON
    [Arguments]    ${response_body}    ${schema_path}
    ${schema}=    Get File    ${schema_path}
    Validate Json Schema    ${response_body}    ${schema}

Gerar Correlation Id
    ${uuid}=    Evaluate    str(__import__('uuid').uuid4())
    RETURN    ${uuid}
```

## kafka_helpers.resource — Keywords Kafka

```robot
*** Settings ***
Library    String
Library    Collections

*** Keywords ***
Aguardar Evento Kafka
    [Arguments]    ${topic}    ${key}    ${timeout}=30s
    [Documentation]    Polling para validar que evento foi publicado no topic
    Wait Until Keyword Succeeds    ${timeout}    2s
    ...    Verificar Evento Publicado    ${topic}    ${key}

Validar Estado Apos Processamento
    [Arguments]    ${endpoint}    ${id}    ${expected_status}    ${timeout}=30s
    [Documentation]    Aguarda processamento assíncrono e valida estado final
    Wait Until Keyword Succeeds    ${timeout}    2s
    ...    Verificar Status Recurso    ${endpoint}    ${id}    ${expected_status}
```

## Template de suite .robot

```robot
*** Settings ***
Documentation
...    Suite: {Nome do Fluxo} - Happy Path
...    Flow de referência: flows/services/{service}/flows/{flow}.md
...    Serviço: {service-name}
...    Atualizado em: {data}

Resource    ../../resources/keywords/common.resource
Resource    ../../resources/keywords/{service}_api.resource
Variables   ../../resources/variables/common.yaml

Suite Setup      Criar Sessao API    ${BASE_URL}    {service}
Suite Teardown   Limpar Dados De Teste

*** Variables ***
${CORRELATION_ID}    ${EMPTY}

*** Test Cases ***
{Nome Do Caso De Teste Descritivo}
    [Documentation]    Descreve o cenário e referencia o step do flow
    [Tags]    happy-path    {service}    {fluxo}
    # Given
    ${CORRELATION_ID}=    Gerar Correlation Id
    # When
    # Then

*** Keywords ***
Limpar Dados De Teste
    [Documentation]    Teardown: remove dados criados durante o teste
```

## Variáveis obrigatórias em common.yaml

```yaml
BASE_URL: ${ENV_BASE_URL}
AUTH_TOKEN: ${ENV_AUTH_TOKEN}
DEFAULT_TIMEOUT: 30s
KAFKA_CONSUMER_TIMEOUT: 60s
```

## Tags padrão
- `happy-path` / `error-case`
- `smoke` — testes mínimos para validar ambiente
- `regression` — suite completa
- `async` — envolve processamento Kafka
- `{service}` — tag do serviço
- `{bounded-context}` — tag do bounded context
