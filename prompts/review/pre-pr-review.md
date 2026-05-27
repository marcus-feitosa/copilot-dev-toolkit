# Prompt: Revisão Pre-PR

## Uso
Invoque antes de abrir qualquer Pull Request para garantir qualidade e sync de specs.

## Prompt

```
Atue como o agente code-reviewer.

Contexto:
- Serviço: {service-name}
- Branch: {nome da branch}
- Issue/ticket relacionado: {id}

Execute a revisão completa:
1. Analise os arquivos modificados nesta branch vs main
2. Verifique violações de arquitetura hexagonal (instructions/java-quarkus.md)
3. Detecte drift entre o código e os flows em flows/services/{service-name}/
4. Apresente relatório classificado (bloqueantes, recomendações, informativos)
5. Proponha atualizações de spec via HITL antes de qualquer escrita

Não aprove se houver violações bloqueantes de arquitetura.
```
