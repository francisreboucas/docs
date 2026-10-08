# Padrões SQL

SQL legível, previsível e seguro. Palavras-chave em maiúsculas. Identificadores em `snake_case`. Nunca concatene entrada de usuário na query: use bind parameters.

## Nomenclatura

| Objeto | Forma | Exemplo |
|---|---|---|
| Tabela | substantivo plural, snake_case | `invoices`, `invoice_items` |
| Coluna | snake_case, singular | `created_at`, `customer_id` |
| Chave primária | `id` | `id` |
| Chave estrangeira | `{tabela_singular}_id` | `customer_id` |
| Índice | `idx_{tabela}_{colunas}` | `idx_invoices_status` |
| Unique | `uq_{tabela}_{colunas}` | `uq_users_email` |
| Constraint | `ck_{tabela}_{regra}` | `ck_invoices_total_positive` |

Tabelas de ligação: `invoice_tags` (os dois plurais em ordem estável). Evite prefixo de tipo (`tbl_`, `fld_`).

Datas e instantes: `*_at` (`created_at`, `issued_at`). Flags booleanas: `is_*` / `has_*`. Moeda em inteiro (centavos) ou `NUMERIC`, nunca `FLOAT`.

## Formatação

Uma cláusula principal por linha. Colunas alinhadas de forma simples, sem arte ASCII. Indentação de 4 espaços.

```sql
SELECT
    invoices.id,
    invoices.status,
    customers.email
FROM invoices
INNER JOIN customers
    ON customers.id = invoices.customer_id
WHERE invoices.status = :status
    AND invoices.created_at >= :from
ORDER BY invoices.created_at DESC
LIMIT :limit
OFFSET :offset;
```

`JOIN` explícito (`INNER`, `LEFT`). Evite join implícito no `WHERE`.

## Tipos e nulos

* Inteiros para IDs.
* `TIMESTAMPTZ` (ou equivalente com fuso) para instantes.
* `DATE` para dia civil.
* `TEXT` ou `VARCHAR` com tamanho justificado.
* `BOOLEAN` para flags. Sem `0`/`1` em coluna de estado binário quando o banco oferece boolean.
* `NOT NULL` por padrão. Null só quando a ausência é um estado real.

## Migrations

Uma migration, uma intenção (criar tabela, adicionar coluna, backfill). Nome: `YYYYMMDDHHMMSS_descricao_curta`. Reversível quando o banco e o dado permitirem.

Não edite migration já aplicada em produção. Crie outra.

Seed e fixture usam `example.com` e dados óbvios de teste.

## Escrita segura

```sql
-- bom: parâmetro ligado
SELECT id, email
FROM users
WHERE email = :email;

-- ruim: interpolação
-- SELECT id FROM users WHERE email = '$email';
```

Transação em volta de writes que precisam ser atômicos. `SELECT … FOR UPDATE` só com transação curta e índice que cubra o filtro.

## Consultas

Prefira colunas explícitas a `SELECT *` em código de aplicação. `*` vale em exploração e em `COUNT(*)`.

Paginação com `LIMIT`/`OFFSET` só em conjuntos pequenos. Para listas grandes, use cursor (`WHERE id > :last_id ORDER BY id`).

Índice para filtro + ordenação reais. Não indexe “por precaução” em coluna de baixa seletividade sem medição.
