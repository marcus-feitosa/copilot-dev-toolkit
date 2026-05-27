# GitHub Copilot — Global Instructions

## Stack de referência
- **Linguagem**: Java 21 (Virtual Threads habilitado)
- **Framework**: Quarkus (RESTEasy Reactive, Panache, SmallRye)
- **Arquitetura**: Hexagonal (Ports & Adapters)
- **Mensageria**: Apache Kafka via AWS MSK
- **Banco de dados**: Aurora PostgreSQL + DynamoDB
- **Infraestrutura**: AWS ECS/Fargate
- **Testes automatizados**: Robot Framework

## Princípios arquiteturais obrigatórios

### Hexagonal
- **Domínio** nunca depende de infraestrutura. Infraestrutura depende do domínio.
- **Ports** são interfaces Java no pacote `domain.port`
- **Adapters** implementam ports e vivem em `infrastructure` (inbound) ou `infrastructure.out` (outbound)
- **Use Cases** vivem em `application.usecase` e orquestram o domínio
- Nunca importar classes de Quarkus, JPA ou Kafka dentro de `domain.*`

### Estrutura de pacotes esperada
```
src/main/java/{group}/{service}/
├── domain/
│   ├── model/          # Entidades e Value Objects
│   ├── port/
│   │   ├── in/         # Driving ports (interfaces de entrada)
│   │   └── out/        # Driven ports (interfaces de saída)
│   └── exception/
├── application/
│   └── usecase/        # Implementações dos driving ports
├── infrastructure/
│   ├── in/
│   │   ├── rest/       # REST adapters (RESTEasy)
│   │   └── messaging/  # Kafka consumers
│   └── out/
│       ├── persistence/ # JPA/Panache adapters
│       ├── messaging/   # Kafka producers
│       └── client/     # HTTP clients (REST Client)
└── config/             # CDI producers, configurações
```

## Convenções de código

- Records Java para Value Objects imutáveis
- `sealed interfaces` para modelar estados e resultados
- Métodos de use case lançam exceções de domínio, nunca exceções de infraestrutura
- Logs estruturados em JSON com MDC (correlationId, traceId obrigatórios)
- Toda operação Kafka usa Transactional Outbox Pattern quando há escrita em banco

## Padrões de nomenclatura
- Use Cases: `{Verbo}{Substantivo}UseCase` (ex: `RegistrarDuplicataUseCase`)
- Ports in: `{Verbo}{Substantivo}Port` (ex: `RegistrarDuplicataPort`)
- Ports out: `{Substantivo}Repository`, `{Substantivo}EventPublisher`
- Adapters REST: `{Recurso}Resource`
- Adapters Kafka: `{Evento}Consumer`, `{Evento}Producer`

## HITL — Human in the Loop
- Nenhuma geração de migration SQL é aplicada sem aprovação explícita
- Nenhuma atualização de spec/flow é commitada sem aprovação explícita
- Toda ação destrutiva exibe resumo e aguarda confirmação antes de executar

## SDD — Specification-Driven Development
- Código novo sempre parte de uma spec (ADR, OpenAPI ou flow .md)
- Se a spec não existir, o agente `spec-writer` deve ser invocado primeiro
- Se o código diverge da spec, o agente `code-reviewer` detecta e propõe sync
