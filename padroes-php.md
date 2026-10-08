# Padrões PHP

Este guia descreve o estilo PHP da equipe. A base normativa é [PSR-1](https://www.php-fig.org/psr/psr-1/), [PSR-4](https://www.php-fig.org/psr/psr-4/), [PSR-12](https://www.php-fig.org/psr/psr-12/) e [PER Coding Style 3.1](https://www.php-fig.org/per/coding-style/). Em caso de dúvida, a especificação PER-CS prevalece.

Nenhum recurso novo entra no repositório sem testes que passem.

## Dicas rápidas

* Abra tags com `<?php`. Omita `?>` em arquivos só de PHP.
* Use UTF-8 sem BOM e quebra de linha LF.
* Indente com 4 espaços. Tabs não indentam código.
* Uma classe, interface, trait ou enum por arquivo.
* `declare(strict_types=1);` no topo de cada arquivo PHP puro.
* Classes em `PascalCase`. Métodos e propriedades em `camelCase`. Constantes em `UPPER_SNAKE_CASE`.
* Visibilidade em toda propriedade, constante e método. Sem prefixo `_` ou `__` para indicar visibilidade.
* Tipos curtos: `int`, `bool`, `float`, `string`. Palavras-chave em minúsculas (`true`, `false`, `null`).
* Autoload PSR-4. Não use `require_once` para classes do projeto.

## Arquivos

Arquivos PHP usam LF, terminam com uma linha não vazia e um único LF. Sem espaço em branco no fim da linha. Uma instrução por linha.

```php
<?php

declare(strict_types=1);

namespace App\Billing;

use App\Billing\Invoice;

final class InvoiceService
{
    public function issue(Invoice $invoice): void
    {
        // ...
    }
}
```

## Cabeçalho do arquivo

Blocos nesta ordem, separados por uma linha em branco:

1. Tag `<?php`
2. Docblock do arquivo (opcional)
3. `declare`
4. `namespace`
5. `use` de classes
6. `use function`
7. `use const`
8. Restante do código

`declare(strict_types=1)` sem espaços dentro dos parênteses. Imports sem barra invertida inicial.

```php
<?php

declare(strict_types=1);

namespace App\User;

use App\Shared\Clock;
use App\User\Exception\UserNotFound;

use function sprintf;

use const DATE_ATOM;
```

## Indentação e linhas

Limite suave de 120 caracteres. Prefira quebrar perto de 80. Sem limite rígido.

Listas em uma linha não levam vírgula final. Listas multilinha levam vírgula final.

```php
function beep(string $a, string $b, string $c): void
{
    // ...
}

function beep(
    string $a,
    string $b,
    string $c,
): void {
    // ...
}
```

## Classes, propriedades e métodos

O termo “classe” inclui interfaces, traits e enums.

Chave de abertura da classe na linha seguinte ao nome. Chave de fechamento na linha após o último membro, sem linha em branco antes.

```php
class UserRepository implements UserRepositoryInterface
{
    public function __construct(
        private readonly Connection $connection,
    ) {}
}
```

`new` sempre com parênteses: `new Foo()`. Classe vazia pode ficar em uma linha: `class NotFoundException extends RuntimeException {}`.

Visibilidade em todas as propriedades, constantes e métodos. Uma propriedade por declaração. Sem `var`.

```php
class Order
{
    public const STATUS_OPEN = 'open';

    private string $id;

    protected Clock $clock;
}
```

Parâmetros com valor padrão ficam no fim da lista. Tipo de retorno com espaço depois dos dois-pontos: `): string`.

```php
public function findByEmail(string $email, bool $includeDeleted = false): ?User
{
    // ...
}
```

## Nomenclatura

| Elemento | Forma | Exemplo |
|---|---|---|
| Namespace / classe | PascalCase | `App\Billing\InvoiceService` |
| Método / propriedade / variável | camelCase | `$invoiceTotal`, `findById()` |
| Constante | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| Arquivo PSR-4 | igual à classe | `src/Billing/InvoiceService.php` |

Siglas tratadas como palavra: `XmlHttpRequest`, `HttpClient`, `UrlParser`.

Variáveis descritivas e curtas. Objetos também em camelCase:

```php
$user = 'John';
$users = ['John', 'Hans', 'Arne'];
$dispatcher = new Dispatcher();
```

## Autoload (PSR-4)

O caminho do arquivo espelha o namespace. Composer carrega as classes.

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

`App\User\UserService` vive em `src/User/UserService.php`.

## Estruturas de controle

Espaço depois da palavra-chave, sem espaço dentro dos parênteses, espaço antes da chave. Sempre use chaves.

```php
if ($expr1) {
    // ...
} elseif ($expr2) {
    // ...
} else {
    // ...
}
```

Use `elseif`, não `else if`. Operadores booleanos no início da linha quando a condição quebra:

```php
if (
    $expr1
    && $expr2
) {
    // ...
}
```

### Ternário e `match`

Ternário só cabe em uma linha. Sem aninhar ternários. Prefira `match` para ramificações por valor.

```php
$variable = $options['variable'] ?? true;

$result = match ($status) {
    'open' => 'Aberta',
    'paid' => 'Paga',
    default => 'Desconhecida',
};
```

### `switch`

```php
switch ($expr) {
    case 0:
        echo 'First';
        break;
    case 1:
        echo 'Second';
        break;
    default:
        echo 'Default';
        break;
}
```

### `for`, `foreach`, `while`, `try`

```php
foreach ($items as $key => $item) {
    // ...
}

try {
    // ...
} catch (ClientException $exception) {
    // ...
} finally {
    // ...
}
```

## Chamadas

Sem espaço entre o nome e `(`. Um espaço depois de cada vírgula. Encadeamento indentado um nível:

```php
$email
    ->from('foo@example.com')
    ->to('bar@example.com')
    ->subject('A great message')
    ->send();
```

Argumentos nomeados: `foo(id: $id, notify: true)`.

## Arrays

Sintaxe curta `[]`.

```php
$item = [
    'name' => 'Ada',
    'role' => 'admin',
];
```

## Closures e arrow functions

```php
$filtered = array_filter($items, function (Item $item) use ($limit): bool {
    return $item->total() > $limit;
});

$ids = array_map(fn (Item $item): int => $item->id(), $items);
```

## Enums e attributes

```php
enum InvoiceStatus: string
{
    case Open = 'open';
    case Paid = 'paid';

    public function isSettled(): bool
    {
        return $this === self::Paid;
    }
}

#[Route('/invoices/{id}', methods: ['GET'])]
final class ShowInvoiceController
{
    // ...
}
```

## Tipos compostos

Sem espaço em volta de `|` e `&`. `null` por último. Prefira `?T` para união simples com null.

```php
function find(int|string $id): ?User
{
    // ...
}

function accept(array|(ArrayAccess&Traversable) $input): void
{
    // ...
}
```

## Comentários

Comentários em português. Explique a decisão, não a linha óbvia. Docblocks com [PHPDoc](https://docs.phpdoc.org/):

```php
/**
 * Emite a fatura e dispara o evento de cobrança.
 *
 * @throws InvoiceAlreadyIssued
 */
public function issue(Invoice $invoice): void
{
    // ...
}
```

Tags úteis: `@param`, `@return`, `@throws`, `@deprecated`, `@see`. Prefira tipos nativos da linguagem quando o PHP já expressa o contrato.

Use `// TODO:` para trabalho pendente e `// FIXME:` para defeito conhecido.

## Operadores

Espaço dos dois lados de operadores binários (`=`, `.`, `===`, `??`, `=>`). Sem espaço no incremento (`$i++`) nem na negação (`!$ready`).

Compare com `===` e `!==`.

## Exemplo completo

```php
<?php

declare(strict_types=1);

namespace App\Billing;

use App\Billing\Exception\InvoiceAlreadyIssued;
use App\Shared\Clock;

final class InvoiceService
{
    public function __construct(
        private readonly InvoiceRepository $invoices,
        private readonly Clock $clock,
    ) {}

    public function issue(int $invoiceId): Invoice
    {
        $invoice = $this->invoices->get($invoiceId);

        if ($invoice->status() !== InvoiceStatus::Open) {
            throw new InvoiceAlreadyIssued($invoiceId);
        }

        $invoice->issue($this->clock->now());
        $this->invoices->save($invoice);

        return $invoice;
    }
}
```
