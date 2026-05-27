# Skill: Summarize Plan

## Objetivo
Apresentar um resumo estruturado do plano de execução de qualquer agente
antes que qualquer ação seja tomada.

## Formato padrão

```
## Plano de Execução — {agente} — {contexto}

### Objetivo
{O que este plano pretende alcançar}

### Pré-condições verificadas
- [x] {condição verificada}
- [ ] {condição não verificada — requer atenção}

### Ações planejadas
{n}. {descrição da ação}
   - Arquivo: {path}
   - Operação: criar | atualizar | deletar
   - Reversível: sim | não

### O que NÃO será feito
- {limitação explícita do escopo}

### Tempo estimado
{estimativa se relevante}

---
✅ Confirma? (sim / não / ajustar escopo)
```

## Regras
- Sempre listar explicitamente o que está FORA do escopo
- Não omitir nenhuma operação de escrita ou execução
- Se houver incerteza em alguma ação, marcar com ⚠️ e explicar
