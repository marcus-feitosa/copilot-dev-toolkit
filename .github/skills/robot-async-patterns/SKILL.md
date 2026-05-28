---
name: robot-async-patterns
description: >
  Padrões Robot Framework para testar fluxos assíncronos com Kafka em serviços Quarkus.
  Cobre os padrões Publish→Poll→Assert, Idempotência e Error Path com Retry Kafka,
  com timeouts recomendados por cenário. Invocada pelo robot-test-generator.
allowed-tools: shell
---

# Skill: Robot Framework — Padrões para Fluxos Assíncronos (Kafka)

## Objetivo
Guia específico para testar fluxos assíncronos com Kafka em serviços Quarkus,
onde o teste precisa aguardar processamento de eventos antes de validar estado.

## Padrão: Publish → Poll → Assert

### Problema
Após publicar um evento ou chamar um endpoint que dispara um fluxo assíncrono,
o estado final só estará disponível após processamento pelo consumer.
Testes síncronos falham por race condition.

### Solução: Wait Until Keyword Succeeds

```robot
*** Test Cases ***
Deve Processar Evento De Registro Com Sucesso
    [Documentation]
    ...    Fluxo: registro-duplicatas/flows/happy-path.md
    ...    Publica evento de registro e valida estado final após processamento
    [Tags]    happy-path    async    registro

    # Given — dados de entrada
    ${payload}=    Criar Payload Registro Duplicata Valido

    # When — dispara o fluxo
    ${response}=    POST Em    /v1/duplicatas    ${payload}
    Status Should Be    202    ${response}
    ${id}=    Get From Dictionary    ${response.json()}    id

    # Then — aguarda processamento assíncrono e valida
    Aguardar E Validar Estado    ${id}    REGISTRADA    timeout=30s

*** Keywords ***
Aguardar E Validar Estado
    [Arguments]    ${id}    ${expected_status}    ${timeout}=30s
    Wait Until Keyword Succeeds    ${timeout}    2s
    ...    Validar Status Duplicata    ${id}    ${expected_status}

Validar Status Duplicata
    [Arguments]    ${id}    ${expected_status}
    ${response}=    GET Em    /v1/duplicatas/${id}
    Status Should Be    200    ${response}
    ${status}=    Get From Dictionary    ${response.json()}    status
    Should Be Equal    ${status}    ${expected_status}
```

## Padrão: Idempotência

```robot
*** Test Cases ***
Deve Ser Idempotente Para Mesmo Evento
    [Documentation]
    ...    Envia o mesmo payload duas vezes e valida que o estado final é idêntico
    [Tags]    idempotencia    {service}

    ${payload}=    Criar Payload Com Id Fixo    id=IDEM-001

    # Primeira chamada
    ${r1}=    POST Em    /v1/duplicatas    ${payload}
    Status Should Be    201    ${r1}

    # Segunda chamada — deve retornar 200 ou 201 sem duplicar dados
    ${r2}=    POST Em    /v1/duplicatas    ${payload}
    Should Be True    ${r2.status_code} in [200, 201]

    # Estado final deve ser o mesmo
    Aguardar E Validar Estado    IDEM-001    REGISTRADA
    Validar Que Existe Apenas Um Registro    IDEM-001
```

## Padrão: Error Path com Retry Kafka

```robot
*** Test Cases ***
Deve Reprocessar Evento Apos Falha Transiente
    [Documentation]
    ...    Simula falha no consumer e valida que o evento é reprocessado
    [Tags]    error-case    async    retry

    # Configura falha no primeiro processamento (via mock/flag de teste)
    Habilitar Falha Transiente No Consumer    vezes=1

    ${payload}=    Criar Payload Registro Duplicata Valido
    POST Em    /v1/duplicatas    ${payload}

    # Aguarda mais tempo para cobrir o retry
    Aguardar E Validar Estado    ${payload}[id]    REGISTRADA    timeout=60s
```

## Timeouts recomendados por cenário
| Cenário | Timeout sugerido |
|---------|-----------------|
| Consumer simples (DB write) | 15s |
| Consumer com downstream HTTP | 30s |
| Consumer com retry (1 falha) | 60s |
| Saga multi-step | 90s |
