# Prompt: Gerar Testes Robot a partir de Flow

## Uso
Invoque para gerar test suites Robot Framework a partir de um flow documentado.

## Prompt

```
Atue como o agente robot-test-generator.

Contexto:
- Serviço: {service-name}
- Flow de referência: flows/services/{service-name}/flows/{flow-name}.md
- Ambiente alvo: {dev | staging}

Gere testes Robot Framework cobrindo:
1. Happy path completo conforme o flow
2. Todos os error paths documentados
3. Idempotência (se aplicável)
4. Fluxo assíncrono Kafka (se aplicável) com polling pattern

Siga as convenções em skills/robot/robot-conventions.md e skills/robot/robot-async-patterns.md.
Apresente o plano de cobertura antes de gerar os arquivos.
```
