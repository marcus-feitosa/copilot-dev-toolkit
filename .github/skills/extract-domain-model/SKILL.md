---
name: extract-domain-model
description: >
  Guia a leitura de código-fonte Java/Quarkus e a extração dos elementos do modelo de
  domínio (entidades, value objects, ports, use cases, eventos Kafka, endpoints REST)
  necessários para gerar domain-overview.md e openapi.yml. Invoque esta skill durante
  a fase de análise de qualquer serviço com arquitetura hexagonal.
allowed-tools: shell
---

# Skill: Extract Domain Model

## Objetivo

Guia a leitura do código-fonte Java/Quarkus e a extração dos elementos do modelo de domínio
necessários para gerar o `domain-overview.md` e o `openapi.yml`.

## Quando usar

Invocada pelo `domain-extractor` no Passo 2 e 3, após a fonte estar disponível em `SOURCE_PATH`.

---

## Ordem de leitura recomendada

Leia nesta sequência — cada arquivo fornece contexto para ler o próximo.

```
1. pom.xml                              → nome do serviço, groupId, dependências
2. README.md                            → contexto de negócio (se existir)
3. src/main/resources/application.properties → nome do serviço, topics Kafka
4. src/main/java/**/domain/model/       → entidades e value objects
5. src/main/java/**/domain/port/in/     → driving ports
6. src/main/java/**/domain/port/out/    → driven ports
7. src/main/java/**/application/usecase/→ use cases
8. src/main/java/**/infrastructure/in/rest/      → REST adapters → OpenAPI
9. src/main/java/**/infrastructure/in/messaging/ → Kafka consumers
10. src/main/java/**/infrastructure/out/messaging/→ Kafka producers
11. src/main/java/**/infrastructure/out/persistence/ → adapters de persistência
```

---

## Como extrair cada elemento

### Nome do serviço

```xml
<!-- pom.xml -->
<groupId>br.com.empresa</groupId>
<artifactId>nome-do-servico</artifactId>  ← este é o service-name
```

Também confirme em `application.properties`:
```properties
quarkus.application.name=nome-do-servico
```

---

### Bounded Context

Derive a partir de:
1. `workspace/domain-map.yml` do toolkit — verifique se o serviço já está registrado
2. GroupId do pom.xml (ex: `br.com.empresa.{bounded-context}.{service}`)
3. Nome do pacote raiz Java

---

### Entidades

**Identificadores**: Classe Java (não record) com um campo tipado de ID.

```java
// Exemplo de entidade
public class Duplicata {
    private final DuplicataId id;   ← ID tipado = entidade
    private String cedente;
    private BigDecimal valor;
    // ...
}
```

**Extraia**:
- Nome da classe
- Pacote completo
- Todos os campos (nome + tipo)
- Construtores, factory methods ou métodos de negócio que expressem invariantes
- `sealed interface` de resultado se houver

---

### Value Objects

**Identificadores**: `record` Java sem campo de ID.

```java
// Exemplo de value object
public record DuplicataId(UUID value) {
    public DuplicataId {
        Objects.requireNonNull(value);  ← invariante
    }
}

public record Cedente(String cpfCnpj, String nome) {
    public Cedente {
        if (cpfCnpj == null || cpfCnpj.isBlank()) throw new ...;
    }
}
```

**Extraia**:
- Nome do record
- Todos os campos com tipos
- Validações presentes no compact constructor

---

### Driving Ports (`domain/port/in`)

```java
public interface RegistrarDuplicataPort {
    RegistrarDuplicataResult registrar(RegistrarDuplicataCommand command);
}
```

**Extraia**:
- Nome da interface
- Método(s) declarado(s)
- Tipo do parâmetro de entrada
- Tipo de retorno
- Use case que implementa (busque `implements {NomePort}` nos use cases)

---

### Driven Ports (`domain/port/out`)

```java
// Repository
public interface DuplicataRepository {
    void salvar(Duplicata duplicata);
    Optional<Duplicata> buscarPorId(DuplicataId id);
}

// Event Publisher
public interface DuplicataEventPublisher {
    void publicarDuplicataRegistrada(DuplicataRegistradaEvent evento);
}
```

**Extraia**:
- Nome da interface
- Tipo: `Repository` (operações CRUD/query) ou `EventPublisher` (publicação de eventos)
- Lista de operações com assinaturas simplificadas

---

### Use Cases (`application/usecase`)

```java
@ApplicationScoped
public class RegistrarDuplicataUseCase implements RegistrarDuplicataPort {

    private final DuplicataRepository repository;       ← driven port injetada
    private final DuplicataEventPublisher publisher;    ← driven port injetada

    @Override
    public RegistrarDuplicataResult registrar(RegistrarDuplicataCommand command) {
        // leia o corpo para derivar o fluxo resumido
    }
}
```

**Extraia**:
- Nome da classe
- Qual port implementa (`implements`)
- Quais driven ports são injetadas (campos privados do tipo interface de port/out)
- Fluxo resumido: leia o método principal e descreva em 1-3 passos

---

### Endpoints REST

```java
@Path("/v1/duplicatas")
@ApplicationScoped
public class DuplicataResource {

    @POST
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON)
    public Response registrar(@Valid RegistrarDuplicataRequest request) { ... }

    @GET
    @Path("/{id}")
    @Produces(MediaType.APPLICATION_JSON)
    public Response buscarPorId(@PathParam("id") UUID id) { ... }
}
```

**Extraia**:
- Path base da classe (`@Path`)
- Cada método: HTTP verb, path relativo, tipo do request body, tipo de resposta
- Anotações de validação (`@Valid`, `@NotNull`, `@Size`, etc.) para enrichment do schema OpenAPI

---

### Eventos Kafka — Consumidos

```java
@ApplicationScoped
public class SolicitacaoRegistroConsumer {

    @Incoming("solicitacao-registro")   ← nome do canal
    public CompletionStage<Void> processar(Message<SolicitacaoRegistroPayload> message) { ... }
}
```

Confirme o topic real em `application.properties`:
```properties
mp.messaging.incoming.solicitacao-registro.topic=duplicatas.solicitacao-registro
```

**Extraia**: nome do canal, topic real, nome do consumer, payload type

---

### Eventos Kafka — Publicados

```java
@ApplicationScoped
public class DuplicataRegistradaProducer implements DuplicataEventPublisher {

    @Channel("duplicata-registrada")
    Emitter<DuplicataRegistradaPayload> emitter;
}
```

Confirme o topic real em `application.properties`:
```properties
mp.messaging.outgoing.duplicata-registrada.topic=duplicatas.registrada
```

**Extraia**: nome do canal, topic real, nome do producer, payload type, condição (quando é publicado)

---

## Geração do OpenAPI 3.1

A partir dos REST adapters extraídos, gere o OpenAPI seguindo esta estrutura:

```yaml
openapi: "3.1.0"
info:
  title: {service-name}
  version: "1.0.0"
  description: {conceito de negócio derivado}

servers:
  - url: /v{n}  # baseado no @Path encontrado

paths:
  {/path}:
    {method}:
      summary: {descrição derivada do nome do método Java}
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/{RequestType}'
      responses:
        '200':
          description: Sucesso
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/{ResponseType}'
        '400':
          description: Dados inválidos
        '422':
          description: Regra de negócio violada
        '500':
          description: Erro interno

components:
  schemas:
    {RequestType}:
      type: object
      properties:
        {campo}:
          type: {tipo-json}
          description: {derivado do nome do campo ou anotação}
      required:
        - {campos com @NotNull ou @NotBlank}
```

**Regras de mapeamento Java → JSON Schema**:
| Java | JSON Schema |
|------|------------|
| `String` | `type: string` |
| `UUID` | `type: string, format: uuid` |
| `BigDecimal` | `type: number` |
| `LocalDate` | `type: string, format: date` |
| `OffsetDateTime` / `Instant` | `type: string, format: date-time` |
| `boolean` | `type: boolean` |
| `int` / `long` | `type: integer` |
| `List<T>` | `type: array, items: $ref T` |
| Enum | `type: string, enum: [...]` |

---

## Quando parar e perguntar ao dev

Ao encontrar qualquer situação abaixo, **interrompa a extração deste elemento e pergunte**
antes de continuar. Não assuma, não pule, não marque como TODO se a resposta pode vir do dev agora.

| Situação | O que perguntar |
|----------|----------------|
| Record que parece entidade (tem campo `id` mas é `record`) | "O tipo `{Nome}` é um record com campo `id`. É uma entidade ou um value object de ID?" |
| Interface em `port/in` com mais de um método | "A port `{NomePort}` tem {n} métodos. Qual é o método principal / operação primária?" |
| Classe em `usecase/` que não implementa nenhuma interface | "O use case `{NomeUseCase}` não implementa uma port. Existe uma port correspondente que não encontrei, ou este use case é uma exceção ao padrão?" |
| Topic Kafka sem correspondência no `application.properties` | "O canal `{canal}` não tem topic mapeado no application.properties. Qual é o nome real do topic?" |
| Endpoint retorna `Response` sem body tipado | "O endpoint `{METHOD} {path}` retorna `Response` genérico. Qual é o payload de sucesso?" |
| Schema de request/response em arquivo separado não encontrado | "O tipo `{Tipo}` é referenciado mas não encontrei sua definição nos arquivos lidos. Ele está em outro módulo ou repo?" |

## Flags de qualidade na extração

Ao encontrar qualquer uma das situações abaixo, registre como observação no domain-overview
**e informe o dev no relatório HITL** (Passo 4 do agente):

Marque como `# TODO: verificar manualmente` apenas o que for **impossível determinar** mesmo
após perguntar ao dev (ex: regras de negócio implícitas não documentadas em nenhum lugar).

- Classe de domínio com import de `javax.persistence`, `io.quarkus` ou `org.apache.kafka` (violação hexagonal)
- Use case sem interface de port correspondente
- Consumer Kafka sem checagem de idempotência (sem acesso a DynamoDB ou cache)
- Outbox Pattern ausente quando há Kafka producer + escrita no banco no mesmo use case
- Port com múltiplos métodos (pode indicar violação de CQS)
