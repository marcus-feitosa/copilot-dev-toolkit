---
name: scaffold-service
description: >
  Gera a estrutura inicial de um novo microserviço Java/Quarkus com arquitetura hexagonal:
  pacotes domain/application/infrastructure, pom.xml, application.properties,
  Dockerfile.jvm e .copilot-toolkit.yml. Invoque ao criar um serviço do zero.
---

# Skill: Scaffold Service

## Objetivo
Gerar a estrutura inicial de um novo microserviço Java/Quarkus com arquitetura hexagonal.

## Inputs esperados
- Nome do serviço (kebab-case, ex: `registro-duplicatas`)
- Group ID Maven (ex: `br.com.b3`)
- Bounded context de referência
- Recursos principais do domínio (ex: Duplicata, Registro)

## Estrutura gerada

```
{service-name}/
├── src/
│   ├── main/
│   │   ├── java/{group}/{service}/
│   │   │   ├── domain/
│   │   │   │   ├── model/
│   │   │   │   ├── port/
│   │   │   │   │   ├── in/
│   │   │   │   │   └── out/
│   │   │   │   └── exception/
│   │   │   ├── application/
│   │   │   │   └── usecase/
│   │   │   ├── infrastructure/
│   │   │   │   ├── in/
│   │   │   │   │   ├── rest/
│   │   │   │   │   └── messaging/
│   │   │   │   └── out/
│   │   │   │       ├── persistence/
│   │   │   │       ├── messaging/
│   │   │   │       └── client/
│   │   │   └── config/
│   │   └── resources/
│   │       ├── application.properties
│   │       ├── application-dev.properties
│   │       └── db/migration/
│   └── test/
│       ├── java/{group}/{service}/
│       │   ├── domain/
│       │   ├── application/
│       │   └── infrastructure/
│       └── resources/
│           └── application-test.properties
├── pom.xml
├── Dockerfile.jvm
├── .copilot-toolkit.yml
└── README.md
```

## Conteúdo gerado por arquivo

### pom.xml — dependências base
```xml
<!-- Quarkus BOM, RESTEasy Reactive, Panache, SmallRye Reactive Messaging,
     Hibernate Validator, MapStruct, Flyway, PostgreSQL JDBC,
     Quarkus Test, REST Assured, Mockito -->
```

### application.properties — configuração base
```properties
quarkus.application.name={service-name}
quarkus.log.format=json
quarkus.log.json.additional-field.service.value={service-name}
# MDC fields: correlationId, traceId (configurados via OpenTelemetry)
%dev.quarkus.datasource.devservices.enabled=true
%test.quarkus.datasource.devservices.enabled=true
```

### .copilot-toolkit.yml — referência ao toolkit
```yaml
service: {service-name}
bounded-context: {bounded-context}
toolkit:
  flows: flows/services/{service-name}/
  instructions: instructions/java-quarkus.md
```

## Output
Estrutura de diretórios e arquivos base gerados, aguardando aprovação HITL.
