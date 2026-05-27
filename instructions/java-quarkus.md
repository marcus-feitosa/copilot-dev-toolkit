# Instruções: Java/Quarkus — Arquitetura Hexagonal

## Stack
- Java 21 com Virtual Threads (`quarkus.virtual-threads.enabled=true`)
- Quarkus 3.x — RESTEasy Reactive, Panache (Active Record ou Repository), SmallRye Reactive Messaging
- MapStruct para mapeamento entre camadas (nunca mapeamento manual em use cases)
- Flyway para migrations
- OpenTelemetry + OTLP para tracing distribuído

## Regras de camadas

### domain/model
- Entidades com identidade: classe com `id` tipado (Value Object de ID)
- Value Objects: `record` Java, imutáveis, sem setter
- Resultados de operações: `sealed interface` com `permits` para sucesso e falhas
- Sem anotações de framework em nenhuma classe de domínio

### domain/port/in (Driving Ports)
- Interface Java pura
- Um método principal por port (Command/Query segregation)
- Nome: `{Verbo}{Substantivo}Port`

### domain/port/out (Driven Ports)
- Interface Java pura
- `{Substantivo}Repository`: operações de persistência
- `{Substantivo}EventPublisher`: publicação de eventos

### application/usecase
- Implementa a driving port correspondente
- Injeta driven ports pelo tipo da interface
- Anotado com `@ApplicationScoped`
- Não tem anotação `@Transactional` (responsabilidade do adapter de persistência)
- Log estruturado obrigatório: entrada, saída, exceções

### infrastructure/in/rest
- Classe anotada com `@Path`, `@ApplicationScoped`
- Injeta a driving port (nunca o use case diretamente)
- Responsável por: validação de input, mapeamento request→domínio, resposta HTTP
- Usa `@Valid` para Bean Validation
- Trata exceções de domínio via `ExceptionMapper`

### infrastructure/in/messaging (Kafka Consumer)
- `@ApplicationScoped` com `@Incoming`
- Responsável por: desserialização, idempotência (check DynamoDB), invocação do use case
- Commit manual de offset após processamento bem-sucedido

### infrastructure/out/persistence
- Implementa driven port de repositório
- Anotado com `@ApplicationScoped`
- `@Transactional` aqui, nunca no use case
- Usa Panache Repository ou Active Record
- MapStruct para mapeamento entidade JPA ↔ modelo de domínio

### infrastructure/out/messaging (Kafka Producer)
- Implementa driven port de event publisher
- `@Outgoing` ou `@Channel` + `Emitter`
- Se há escrita em banco na mesma operação: usar Outbox Pattern

## Padrão de log estruturado

```java
// MDC configurado no adapter de entrada (REST ou Kafka consumer)
MDC.put("correlationId", correlationId);
MDC.put("traceId", traceId);
MDC.put("service", "nome-do-servico");

// Log no use case
log.infof("Iniciando registro de duplicata: cedente=%s, valor=%s",
    command.cedente(), command.valor());
```

## Virtual Threads + JDBC
- Virtual Threads habilitado globalmente
- Pool JDBC: mínimo 10, máximo 30 (evitar starvation)
- Operações JDBC sempre anotadas com `@Blocking` se em contexto reativo

## Testes
- Use Cases: JUnit 5 puro + Mockito (`@ExtendWith(MockitoExtension.class)`)
- REST Adapters: `@QuarkusTest` + REST Assured
- Kafka: `@QuarkusTest` com `@TestProfile` + in-memory broker
- Robot Framework: integração e E2E (ver `skills/robot/`)
