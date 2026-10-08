# Padrões de segurança

Segurança é regra de código, não checklist de release. Este guia cobre o dia a dia fullstack. Complementa [HTTP/API](padroes-http-api.md) e [SQL](padroes-sql.md).

## Segredos

Senhas, tokens, chaves e connection strings ficam em variáveis de ambiente ou cofre. Repositório leva `.env.example` sem valores reais. Rotacione o que vazar. Nunca commite dump, `.pem`, `.p12` ou backup.

Logs não registram senha, cookie de sessão, cartão ou token completo. Prefira id de correlação.

## Autenticação e sessão

HTTPS em todo tráfego fora de loopback. Cookies de sessão: `HttpOnly`, `Secure`, `SameSite=Lax` (ou `Strict` se o fluxo permitir). Senha: hash com algoritmo adaptativo (`password_hash` / Argon2id). Timing-safe compare em tokens.

Bloqueio ou backoff após falhas de login. Mensagem genérica: “e-mail ou senha inválidos”.

## CSRF, XSS, SQL

* Formulários e cookies de sessão: token CSRF verificado no servidor.
* APIs Bearer: CSRF não se aplica da mesma forma; ainda assim valide `Origin`/`Content-Type` em endpoints sensíveis de browser.
* SQL só com bind parameters. Veja [Padrões SQL](padroes-sql.md).
* HTML de usuário nunca entra com concatenação. Escape na camada de template. No JS, `textContent` em vez de `innerHTML` para dado externo.

```javascript
titleElement.textContent = invoice.title;
```

Content-Security-Policy sem `unsafe-inline` quando o app permitir. `X-Content-Type-Options: nosniff`. `Referrer-Policy` restritiva.

## Autorização

Autenticação responde “quem é”. Autorização responde “pode isto neste recurso”. Cheque no servidor, em todo endpoint, com o id da sessão — não com campo enviado pelo cliente (`isAdmin=true`).

IDs na URL não bastam: `GET /api/v1/invoices/inv_01J8` confirma que a fatura pertence ao ator.

## Upload e arquivos

Lista de extensões e MIME permitidos. Tamanho máximo. Nome de arquivo gerado pelo servidor. Armazene fora da raiz pública ou sirva via handler. Não execute conteúdo enviado. Imagens reprocessadas, não só renomeadas.

## Dependências e headers

Atualize PHP, Node e pacotes com CVE conhecido. `composer audit` / `npm audit` no CI. Não deixe stack trace em produção (`display_errors=Off`). Erro 500 genérico para o cliente; detalhe no log.

Rate limit em login, reset de senha e APIs públicas. CORS explícito: origens nomeadas, não `*` com credenciais.

## Dados pessoais

Colete o mínimo. Mascarar PII em admin lists quando o operador não precisar do valor completo. Direito de exclusão e exportação passam pela aplicação, não por delete solto no banco de log.

## Checklist rápido de PR

* Entrada validada no servidor
* Query parametrizada
* Saída escapada
* Autorização no recurso
* Segredo fora do diff
* Erro sem vazamento interno
