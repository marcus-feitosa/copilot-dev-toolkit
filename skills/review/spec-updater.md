# Skill: Spec Updater

## Objetivo
Propor e aplicar atualizações nos flows documentados a partir dos drifts
detectados pelo `spec-drift-detector`. Nunca atualiza arquivos sem aprovação HITL.

## Processo

1. Receba o relatório de drift do `spec-drift-detector`
2. Para cada drift de recomendação identificado:
   - Leia o flow atual em `flows/services/{service}/flows/{flow}.md`
   - Gere a versão atualizada do flow com as mudanças necessárias
   - Apresente o diff proposto (antes vs depois)
3. Apresente todas as propostas ao dev via HITL (uma por vez ou em bloco)
4. Somente após aprovação explícita: aplique as atualizações
5. Adicione entrada no `## Changelog` do flow atualizado

## Formato do diff proposto

```
### Atualização proposta: flows/services/{service}/flows/{flow}.md

**Motivo**: {descrição do drift detectado}
**PR de referência**: #{n}

--- Antes
{trecho atual do flow}

+++ Depois
{trecho atualizado}
```

## Regras de atualização
- Preserve o histórico — nunca delete seções, apenas atualize
- Mantenha o template padrão do flow
- Adicione sempre a seção `## Changelog` se não existir
- Formato de entrada do changelog: `{data} — PR #{n}: {descrição da mudança}`

## Output
Arquivos `.md` atualizados em `flows/services/{service}/flows/`,
somente após aprovação HITL explícita.
