# Flow: {Nome do Fluxo}

**Serviço**: {service-name}
**Bounded Context**: {bounded-context}
**Status**: Proposto | Aprovado | Deprecated
**Versão**: 1.0
**Data**: {YYYY-MM-DD}
**Autor**: {nome}

---

## Contexto
{Descreva brevemente o propósito deste fluxo e qual necessidade de negócio ele atende}

## Atores / Sistemas envolvidos
- **Iniciador**: {quem ou o que dispara o fluxo}
- **Serviços internos**: {lista de serviços envolvidos}
- **Dependências externas**: {APIs externas, Bacen, etc.}

---

## Happy Path

### Pré-condições
- {condição necessária para o fluxo iniciar}

### Sequência
```
{Iniciador}
    │
    ├──[1] {descrição da ação 1}
    │       └── {resultado esperado}
    │
    ├──[2] {descrição da ação 2}
    │       └── {resultado esperado}
    │
    └──[n] {descrição da ação final}
            └── Estado final: {estado do sistema}
```

### Estado final esperado
- {descrição do estado do sistema após execução bem-sucedida}

---

## Error Paths

### EP-01: {Nome do Erro}
- **Trigger**: {o que causa este erro}
- **Comportamento**: {o que o sistema faz}
- **Estado final**: {estado do sistema após o erro}
- **Retry**: {sim/não, quantas vezes, intervalo}

### EP-02: {Nome do Erro}
- **Trigger**: {o que causa este erro}
- **Comportamento**: {o que o sistema faz}
- **Estado final**: {estado do sistema após o erro}

---

## Contratos

### Entrada
{Referência ao OpenAPI ou descrição do payload de entrada}

### Eventos publicados
| Topic | Condição | Schema |
|-------|----------|--------|
| {topic} | {quando é publicado} | {link para schema} |

### Eventos consumidos
| Topic | De qual serviço | Schema |
|-------|----------------|--------|
| {topic} | {serviço origem} | {link para schema} |

---

## Decisões técnicas relacionadas
- ADR-{n}: {título do ADR relacionado}

## Casos de teste de referência
- `tests/robot/suites/{service}/{flow}_happy_path.robot`
- `tests/robot/suites/{service}/{flow}_error_cases.robot`

---

## Changelog
| Data | PR | Descrição |
|------|----|-----------|
| {YYYY-MM-DD} | #{n} | Versão inicial |
