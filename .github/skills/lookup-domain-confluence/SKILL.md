---
name: lookup-domain-confluence
description: >
  Busca uma página de domínio no Confluence pelo nome, sob o parent page ID 123456.
  Retorna o ID da página raiz do domínio e suas páginas filhas (por serviço + flow E2E).
  Em caso de domínio não encontrado, orienta o dev a re-informar ou executar domain-extractor.
allowed-tools: confluence_search, confluence_get_page
---

Skill reutilizável para localizar documentação de domínio no Confluence.
Invocada por agents que dependem do output do domain-extractor.

## Entrada

Variável `{domain-name}` fornecida pelo agente chamador (string do nome do domínio,
ex: "Registro", "Escrituração", "Pagamentos").

---

## Passo 1 — Busca no Confluence

Execute:
```
confluence_search(
  query: 'title="{domain-name}"',
  space: "{CONFLUENCE_SPACE_KEY}",
  limit: 10
)
```

Filtre client-side os resultados: manter apenas páginas cujo `ancestor_id` inclua `123456`
(página pai direta ou ancestral).

Se a busca retornar mais de um resultado válido sob 123456, apresente a lista ao dev
e aguarde a escolha antes de continuar.

---

## Passo 2 — Caminho encontrado

Quando uma página de domínio é localizada:

1. Defina `DOMAIN_PAGE_ID = {id encontrado}`
2. Liste as páginas filhas:
   ```
   confluence_search(query: 'parent={DOMAIN_PAGE_ID}', limit: 50)
   ```
3. A partir dos filhos, classifique:
   - `SERVICE_PAGES` — filhos com título prefixado `[Domain]`
   - `OPENAPI_PAGES` — filhos com título prefixado `[OpenAPI]`
   - `CROSS_SERVICE_FLOW_PAGE` — filho com título `[Flow E2E] {domain-name}` (null se não existir)

4. Retorne ao agente chamador:
   ```
   DOMAIN_FOUND = true
   DOMAIN_PAGE_ID = {id}
   DOMAIN_PAGE_TITLE = {title}
   SERVICE_PAGES = [{ id, title, service_name (extraído do título) }, ...]
   OPENAPI_PAGES = [{ id, title, service_name }, ...]
   CROSS_SERVICE_FLOW_PAGE = { id, title } | null
   ```

---

## Passo 3 — Caminho não encontrado

Quando nenhuma página é localizada, apresente ao dev:

```
❓ Domínio não encontrado

O domínio "{domain-name}" não foi localizado no Confluence
(buscado sob a página pai ID 123456).

Opções:
a) Informar um nome diferente (pode haver variação de capitalização ou grafia)
b) Executar o agente domain-extractor para criar a documentação do domínio
c) Cancelar esta operação

Aguardando sua resposta para continuar.
```

O agente chamador deve tratar a resposta:
- `a` → reinvocar esta skill com o novo nome informado
- `b` → parar e orientar: "Execute `/domain-extractor` primeiro para documentar o domínio, depois retorne aqui."
- `c` → abortar a operação atual

---

## Casos de erro

| Situação | Ação |
|----------|------|
| Múltiplas páginas com mesmo título sob 123456 | Apresentar lista ao dev e aguardar escolha |
| Erro de rede / Confluence indisponível | Informar o dev e pausar |
| Página encontrada mas sem filhos | Retornar DOMAIN_FOUND=true com SERVICE_PAGES=[] e CROSS_SERVICE_FLOW_PAGE=null |
