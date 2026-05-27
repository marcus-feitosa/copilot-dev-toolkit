# Instruções Gerais — Plataforma de Desenvolvimento

## Princípios transversais

### Domain-Driven Design
- Bounded Contexts têm linguagem ubíqua própria — não reutilize termos entre contextos sem mapeamento explícito
- Agregados definem fronteiras de consistência — nunca acesse internamente o agregado de outro bounded context
- Anti-Corruption Layer obrigatória entre bounded contexts

### Event-Driven Architecture
- Eventos representam fatos do passado: `DuplicataRegistrada`, não `RegistrarDuplicata`
- Consumidores são idempotentes — verificação de processamento anterior obrigatória
- Schema de eventos versionado — nunca quebre compatibilidade retroativa
- Dead Letter Queue configurada para todos os consumers

### Observabilidade (obrigatório em todos os serviços)
- W3C Trace Context propagado em todos os requests HTTP e eventos Kafka
- `correlationId` e `traceId` no MDC de todos os logs
- Logs em JSON estruturado (sem log em texto livre em produção)
- Métricas Micrometer expostas em `/q/metrics`

### Segurança
- Nunca logar dados sensíveis (CPF, CNPJ, valores financeiros sem mascaramento)
- Secrets via AWS Secrets Manager — nunca em application.properties versionado
- TLS obrigatório em toda comunicação inter-serviços

### Git e PRs
- Branch: `feature/{ticket-id}-{descricao-curta}`
- Commit: Conventional Commits (`feat:`, `fix:`, `docs:`, `refactor:`)
- PR sem spec aprovada → bloqueado pelo `code-reviewer`
- PR com violação de arquitetura bloqueante → não pode ser mergeado

## Workflow SDD obrigatório
```
Issue/Requisito
    ↓
spec-writer → ADR / OpenAPI / Flow (.md)
    ↓ HITL: dev aprova spec
harness-runner → implementação + testes unitários
    ↓
robot-test-generator → testes de integração/E2E
    ↓ HITL: dev revisa testes gerados
code-reviewer → qualidade + sync de spec
    ↓ HITL: dev aprova relatório e atualizações
PR aberto
```
