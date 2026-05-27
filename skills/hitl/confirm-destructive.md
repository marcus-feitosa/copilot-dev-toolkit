# Skill: Confirm Destructive Action

## Objetivo
Apresentar checkpoint obrigatório antes de qualquer ação destrutiva ou irreversível.

## Ações que requerem esta skill
- Deleção de arquivos
- Migrations SQL com DROP ou ALTER de colunas existentes
- Sobrescrita de flows aprovados
- Execução de comandos de terminal destrutivos

## Formato de apresentação

```
⚠️  AÇÃO DESTRUTIVA DETECTADA

Agente: {nome do agente}
Ação: {descrição precisa do que será feito}
Alvo: {arquivo, tabela, recurso afetado}
Reversível: {sim (via git) / não}

Impacto estimado:
- {consequência 1}
- {consequência 2}

---
Digite CONFIRMO para prosseguir ou qualquer outra coisa para cancelar.
```

## Regras
- Nunca interpretar resposta ambígua como confirmação
- Aceitar apenas "CONFIRMO" (maiúsculas) como aprovação
- Qualquer outra resposta cancela a ação e retorna ao agente com status `cancelado`
