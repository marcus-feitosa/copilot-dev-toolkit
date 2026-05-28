---
name: write-test-suite
description: >
  Gera testes unitários e de integração para componentes Java/Quarkus seguindo
  a estrutura hexagonal. Cobre camadas domain (JUnit puro), application (Mockito)
  e infrastructure (QuarkusTest + REST Assured / Kafka). Invoque via harness-runner.
---

# Skill: Write Test Suite (JUnit/Quarkus)

## Objetivo
Gerar testes unitários e de integração para componentes Java/Quarkus seguindo
a estrutura hexagonal.

## Estratégia por camada

### Domain (testes unitários puros)
- Sem mocks de framework, sem anotações Quarkus
- Testa regras de negócio isoladas do domínio
- Usa JUnit 5 puro + AssertJ
- Cobre: criação de entidades, value objects, exceções de domínio

```java
class DuplicataTest {
    @Test
    void deveLancarExcecaoQuandoValorNegativo() {
        assertThatThrownBy(() -> new Duplicata(/* valor negativo */))
            .isInstanceOf(ValorInvalidoException.class)
            .hasMessageContaining("valor deve ser positivo");
    }
}
```

### Application (testes unitários com mocks)
- Testa Use Cases isolados
- Mocks das driven ports via Mockito
- Verifica orquestração e regras de aplicação
- Usa `@ExtendWith(MockitoExtension.class)`

```java
@ExtendWith(MockitoExtension.class)
class RegistrarDuplicataUseCaseTest {
    @Mock DuplicataRepository repository;
    @Mock DuplicataEventPublisher publisher;
    @InjectMocks RegistrarDuplicataUseCase useCase;

    @Test
    void deveRegistrarDuplicataComSucesso() { ... }

    @Test
    void deveLancarExcecaoQuandoDuplicataJaExiste() { ... }
}
```

### Infrastructure/REST (testes de integração)
- Usa `@QuarkusTest` + REST Assured
- Sobe contexto Quarkus com DevServices
- Testa adapter REST completo

```java
@QuarkusTest
class DuplicataResourceTest {
    @Test
    void deveRetornar201AoCriarDuplicata() {
        given()
            .contentType(ContentType.JSON)
            .body(/* payload */)
        .when()
            .post("/v1/duplicatas")
        .then()
            .statusCode(201)
            .header("Location", matchesPattern(".*/v1/duplicatas/.*"));
    }
}
```

### Infrastructure/Kafka (testes de integração)
- Usa `@QuarkusTest` com `@TestProfile` configurado para in-memory broker
- Testa publish e consume de eventos
- Valida estado final no banco após processamento assíncrono

## Cobertura mínima esperada
- Use Cases: 100% dos caminhos documentados no flow (happy + error paths)
- REST Adapters: todos os status codes documentados no OpenAPI
- Kafka: publish com sucesso + falha de consumo + retry

## Output
Arquivos de teste no path `src/test/java/` do serviço correspondente.
