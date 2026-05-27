# Skill: Code Quality Checklist (Java/Quarkus Hexagonal)

## Objetivo
Avaliar qualidade do código Java/Quarkus submetido em PR, com foco em
arquitetura hexagonal e padrões da plataforma.

## Checklist de arquitetura hexagonal

### 🔴 Bloqueantes (impedem merge)
- [ ] Classe em `domain.*` importa Quarkus, JPA, Kafka ou qualquer framework
- [ ] Use Case instancia diretamente um adapter (violação de inversão de dependência)
- [ ] Adapter de infraestrutura contém lógica de negócio
- [ ] Driven port não tem interface — adapter concreto injetado diretamente

### 🟡 Recomendações
- [ ] Use Case sem teste unitário correspondente
- [ ] Método com mais de uma responsabilidade (> 20 linhas, múltiplos `and` no nome)
- [ ] Exceção de infraestrutura propagada sem ser convertida em exceção de domínio
- [ ] Magic strings/numbers sem constante nomeada

### Observabilidade
- [ ] Log de entrada de use case sem correlationId no MDC
- [ ] Log de entrada de use case sem traceId no MDC
- [ ] Operação Kafka com escrita em banco sem Outbox Pattern

### Quarkus específico
- [ ] `@Transactional` em use case (deve estar no adapter de persistência)
- [ ] `@Inject` em classe de domínio
- [ ] Blocking I/O em contexto reativo sem anotação `@Blocking`
- [ ] Uso de `Thread.sleep()` em vez de Uni/Multi reativo

### Testes
- [ ] Teste de use case usando `@QuarkusTest` (desnecessário para unitário puro)
- [ ] Assertions sem mensagem descritiva em falhas de negócio
- [ ] Ausência de teste para error paths documentados no flow

## Output
Lista classificada de issues entregue ao `code-reviewer` para apresentação HITL.
