# Padrões HTTP e API

APIs HTTP JSON, recursos no plural, erros previsíveis. Versionamento na URL quando a quebra for pública (`/api/v1`).

## URLs e métodos

| Intenção | Método | URL | Status de sucesso |
|---|---|---|---|
| Listar | `GET` | `/api/v1/invoices` | `200` |
| Detalhe | `GET` | `/api/v1/invoices/{id}` | `200` |
| Criar | `POST` | `/api/v1/invoices` | `201` + `Location` |
| Substituir | `PUT` | `/api/v1/invoices/{id}` | `200` |
| Alterar parcial | `PATCH` | `/api/v1/invoices/{id}` | `200` |
| Remover | `DELETE` | `/api/v1/invoices/{id}` | `204` |
| Ação | `POST` | `/api/v1/invoices/{id}/issue` | `200` ou `202` |

Recursos em kebab-case no path: `/api/v1/invoice-items`. IDs opacos na URL. Filtros, ordenação e paginação na query: `?status=open&page=2&per_page=20`.

`GET` e `HEAD` não alteram estado. `PUT` é idempotente. `PATCH` descreve a diferença.

## JSON

`Content-Type: application/json; charset=utf-8`. Chaves em `camelCase`. Datas em ISO-8601 UTC (`2026-10-08T13:45:00Z`).

```json
{
  "id": "inv_01J8",
  "status": "open",
  "customerId": "cus_19",
  "totalCents": 12990,
  "createdAt": "2026-10-08T13:45:00Z"
}
```

Coleções envelopadas quando há paginação:

```json
{
  "data": [],
  "meta": {
    "page": 1,
    "perPage": 20,
    "total": 135
  }
}
```

Resposta `201` inclui `Location: /api/v1/invoices/inv_01J8`. `204` não tem corpo.

## Erros

Corpo estável, mensagem segura para o cliente, código máquina para o app.

```json
{
  "error": {
    "code": "invoice_already_issued",
    "message": "A fatura já foi emitida.",
    "details": [
      {
        "field": "status",
        "message": "Esperado open, recebido paid."
      }
    ]
  }
}
```

| Situação | Status |
|---|---|
| JSON inválido / validação | `400` |
| Sem autenticação | `401` |
| Autenticado sem permissão | `403` |
| Recurso inexistente | `404` |
| Conflito de estado | `409` |
| Rate limit | `429` |
| Falha inesperada | `500` |

Não vaze stack trace, SQL ou caminho de arquivo. Log interno leva o id de correlação (`X-Request-Id`), ecoado no response.

## Autenticação e headers

HTTPS em todo ambiente que não seja loopback. Token no `Authorization` (`Bearer` ou o esquema do projeto). CSRF nas sessões cookie: veja [Padrões de segurança](padroes-seguranca.md).

Headers úteis: `Accept`, `Content-Type`, `Idempotency-Key` em `POST` de criação que o cliente pode repetir.

## Versionamento e compatibilidade

Quebra de contrato (renomear campo, mudar tipo, remover recurso) gera versão nova. Campos novos são aditivos e opcionais. Deprecação anunciada no changelog e, se possível, no header `Deprecation`.

## Cliente (JS)

`fetch` com checagem de `response.ok`. Timeout e abort com `AbortController`. Não ignore `204`. Trate `401` no limite da aplicação (renovar sessão ou redirecionar), não em cada helper.
