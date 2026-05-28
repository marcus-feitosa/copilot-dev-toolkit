---
name: generate-migration
description: >
  Gera scripts de migration Flyway auditáveis e seguros para Aurora PostgreSQL.
  Aplica versionamento correto, padrões de segurança (IF EXISTS, CONCURRENTLY)
  e checklists HITL obrigatórios antes de qualquer escrita. Use ao criar ou
  alterar schemas em serviços Java/Quarkus.
allowed-tools: shell
---

# Skill: Generate Migration

## Objetivo
Gerar scripts de migration Flyway auditáveis e seguros para Aurora PostgreSQL.

## Inputs esperados
- Descrição da mudança de schema necessária
- Nome do serviço e schema de referência
- Contexto: nova feature, refactoring ou correção

## Regras obrigatórias

### Versionamento
- Padrão: `V{n}__{descricao_snake_case}.sql`
- Exemplo: `V3__add_status_column_to_duplicata.sql`
- Nunca reutilizar número de versão
- Nunca usar `flyway.repair` em produção

### Segurança do script
- Sempre usar `IF NOT EXISTS` / `IF EXISTS` nas operações
- Nunca fazer DROP sem backup explícito documentado
- Alterações de coluna: prefer ADD + migrate data + DROP old (em migrations separadas)
- Índices: criar com `CONCURRENTLY` para não bloquear tabela em produção

### Template de migration
```sql
-- Migration: V{n}__{descricao}.sql
-- Serviço: {service-name}
-- Data: {YYYY-MM-DD}
-- Descrição: {o que esta migration faz e por quê}
-- Issue/ADR de referência: {link}

-- =============================================
-- UP
-- =============================================

{sql da migration}

-- =============================================
-- ROLLBACK MANUAL (não executado automaticamente)
-- =============================================
-- {sql de rollback correspondente, documentado para uso manual}
```

### Checklist pre-aprovação HITL
- [ ] Script é idempotente?
- [ ] Rollback manual documentado?
- [ ] Impacto em produção avaliado (lock de tabela, volume de dados)?
- [ ] Índice criado com CONCURRENTLY se tabela > 10k rows?
- [ ] Testado em ambiente de dev com dados reais?

## Output
Arquivo SQL no path `src/main/resources/db/migration/` do serviço de destino,
aguardando aprovação HITL **obrigatória** antes de qualquer escrita.
