# Domain Overview: {service-name}

**Serviço**: {service-name}
**Artefato**: `{groupId}.{artifactId}`
**Bounded Context**: {bounded-context}
**Status**: Gerado | Revisado | Aprovado
**Versão**: 1.0
**Data**: {YYYY-MM-DD}
**Gerado por**: domain-extractor

---

## Conceito de Negócio

{Descrição do propósito de negócio do serviço — o que ele faz, por que existe,
qual problema de domínio resolve e qual é o seu lugar no bounded context.}

---

## Glossário

Termos do domínio identificados no código-fonte do serviço.

| Termo | Definição |
|-------|-----------|
| {Termo} | {Definição derivada do contexto de uso no código} |

---

## Modelo de Domínio

### Entidades

Entidades são objetos com identidade própria e ciclo de vida.

#### {NomeDaEntidade}

- **Pacote**: `{fully.qualified.package.domain.model}`
- **ID**: `{TipoDoId}` — {descrição do ID, ex: UUID gerado na criação}
- **Campos**:

  | Campo | Tipo Java | Descrição |
  |-------|-----------|-----------|
  | `{campo}` | `{Tipo}` | {descrição derivada do uso no código} |

- **Invariantes**:
  - {Regra de negócio observada no construtor, factory method ou método de domínio}

---

### Value Objects

Value Objects são imutáveis, sem identidade própria — definidos pelos seus valores.

#### {NomeDoValueObject}

- **Pacote**: `{fully.qualified.package.domain.model}`
- **Campos**:

  | Campo | Tipo Java | Descrição |
  |-------|-----------|-----------|
  | `{campo}` | `{Tipo}` | {descrição} |

- **Validações**: {Validações presentes no record — ex: Objects.requireNonNull, regex, range}

---

## Ports

### Driving Ports — Entrada (`domain/port/in`)

Interfaces que definem o que o domínio expõe para o mundo externo.

| Interface | Método principal | Input | Output | Implementada por |
|-----------|-----------------|-------|--------|-----------------|
| `{NomePort}` | `{metodo}(...)` | `{InputType}` | `{OutputType}` | `{NomeUseCase}` |

### Driven Ports — Saída (`domain/port/out`)

Interfaces que definem o que o domínio precisa do mundo externo.

| Interface | Tipo | Operações |
|-----------|------|-----------|
| `{NomeRepository}` | Repository | `{metodo1}`, `{metodo2}` |
| `{NomeEventPublisher}` | EventPublisher | `{metodo1}` |

---

## Use Cases

| Use Case | Port implementada | Driven ports usadas | Fluxo resumido |
|----------|------------------|---------------------|----------------|
| `{NomeUseCase}` | `{NomePort}` | `{Repository}`, `{Publisher}` | {1-2 linhas descrevendo o que o use case faz} |

---

## Eventos Kafka

### Consumidos

| Topic | Consumer | Descrição |
|-------|----------|-----------|
| `{topic.name}` | `{NomeConsumer}` | {o que este evento representa e o que o serviço faz ao recebê-lo} |

### Publicados

| Topic | Producer | Condição de publicação |
|-------|----------|----------------------|
| `{topic.name}` | `{NomeProducer}` | {quando este evento é publicado} |

---

## Contrato REST

> O contrato completo está em `openapi.yml` neste diretório.

### Endpoints

| Método | Path | Descrição | Autenticação |
|--------|------|-----------|--------------|
| `{METHOD}` | `{/path}` | {descrição do endpoint} | {Bearer / API Key / Nenhuma} |

---

## Dependências externas

| Dependência | Tipo | Propósito |
|-------------|------|-----------|
| `{nome}` | `{HTTP Client / Kafka / Aurora / DynamoDB}` | {para que é usado} |

---

## Decisões técnicas relacionadas

- ADR-{n}: {título do ADR relevante — se existir}

---

## TODO / Pontos para revisão manual

- [ ] {Item que não foi possível derivar automaticamente do código}

---

## Changelog

| Data | Origem | Descrição |
|------|--------|-----------|
| {YYYY-MM-DD} | domain-extractor | Geração inicial a partir do código-fonte |
