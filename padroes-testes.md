# Padrões de testes

Nenhum recurso novo entra no repositório sem testes que passem. O teste descreve o comportamento observável, não a implementação.

## Pirâmide

| Camada | O que cobre | Exemplo |
|---|---|---|
| Unidade | Regra pura, sem I/O | cálculo de total, parser, policy |
| Integração | Persistência, HTTP client, fila | repositório + banco de teste |
| Contrato / HTTP | Status, JSON, autenticação | `POST /api/v1/invoices` |
| UI pontual | Fluxo crítico no browser | emitir fatura no formulário |

Muitos testes de unidade, menos de integração, poucos de UI. Não substitua um teste de regra por um clique em tela.

## Nomenclatura

Arquivo: `{Unidade}Test.php` ou `{unidade}.test.js`. Nome do caso em português, comportamento + condição.

```php
public function testIssueRecusaFaturaJaEmitida(): void
{
    // ...
}
```

```javascript
test('formatMoney formata centavos em real brasileiro', () => {
  // ...
});
```

## Estrutura AAA

Arrange, Act, Assert. Um comportamento por teste. Vários asserts só quando descrevem o mesmo fato (status + corpo).

```php
public function testIssueMarcaFaturaComoPaga(): void
{
    $invoice = InvoiceFactory::open();
    $service = $this->makeService();

    $issued = $service->issue($invoice->id());

    self::assertSame(InvoiceStatus::Paid, $issued->status());
}
```

## Dados

Factories e builders. Sem copiar linhas de SQL no teste de unidade. Bancos de integração: schema migrado, transação revertida (ou database dedicada). Horário injetável (`Clock`), não `new DateTimeImmutable()` escondido na regra.

E-mails e URLs: `user@example.com`. Segredos: valores falsos óbvios (`test-token`).

## Isolamento

Testes independentes. Ordem de execução irrelevante. Sem dependência de rede pública. HTTP externo em double (`fake`, `mock` no limite). Relógio, random e UUID injetados quando a regra depende deles.

Não teste métodos `private`. Teste o efeito público. Se isso for difícil, o desenho da unidade precisa de costura, não de reflexão.

## O que não testar

* Getters sem lógica
* Código gerado
* Framework em si
* CSS visual pixel a pixel, salvo regressão já sofrida

## Frontend

Módulos JS: teste funções puras e o parser de resposta. DOM: teste o listener com `fetch` falsificado. Evite snapshot gigante de markup.

## Falha útil

Mensagem de assert diz o esperado. Logs de teste não imprimem PII. Falhou na CI = vermelho no PR, mesmo que “passe na minha máquina”.
