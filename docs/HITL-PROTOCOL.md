# HITL Protocol — Human in the Loop

## Princípio
Nenhum agente escreve, deleta ou executa comandos sem aprovação explícita do dev.
Silêncio nunca é interpretado como confirmação.

## Níveis de aprovação

| Nível | Quando | Formato |
|-------|--------|---------|
| **Informativo** | Leitura e análise apenas | Sem aprovação necessária |
| **Plano** | Antes de qualquer escrita | `sim / não / ajustar` |
| **Destrutivo** | Deleção, migrations DROP | Digitar `CONFIRMO` explicitamente |

## Fluxo padrão de aprovação

```
Agente apresenta plano
    ↓
Dev responde: sim | não | ajustar
    ↓
sim → agente executa
não → agente cancela, retorna controle ao dev
ajustar → agente recolhe ajustes e reapresenta plano
```

## O que NUNCA pode ser feito sem HITL
- Escrever ou sobrescrever qualquer arquivo `.md` de flow
- Gerar ou executar migrations SQL
- Criar ou modificar arquivos de configuração de produção
- Qualquer `git commit`, `git push` ou operação de terminal

## Responsabilidade do dev
O HITL não substitui o julgamento do dev — ele garante que o dev está no controle.
A aprovação é do dev, não do agente.
