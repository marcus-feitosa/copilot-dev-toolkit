# Prompt: Spec a partir de User Story

## Uso
Invoque este prompt para transformar uma user story ou issue em spec técnica antes de implementar.

## Prompt

```
Atue como o agente spec-writer.

Contexto do serviço:
- Serviço: {service-name}
- Bounded Context: {bounded-context}
- Flows existentes: flows/services/{service-name}/

User Story / Issue:
{cole aqui a descrição da issue ou user story}

Fluxo esperado:
1. Verifique o domain-map.yml para confirmar o serviço correto
2. Verifique se já existe flow relacionado que deva ser atualizado
3. Gere: [ADR se houver decisão técnica relevante] + [Flow .md] + [OpenAPI se houver endpoint novo]
4. Apresente tudo para aprovação antes de salvar

Siga as instruções em instructions/java-quarkus.md e os templates em flows/services/_template/.
```
