# Padrões JavaScript

Guia de estilo para JavaScript ES2020+ em módulos nativos (ESM). Sem jQuery. Sem `var`.

## Índice

1. [Módulos](#módulos)
2. [Tipos e declarações](#tipos-e-declarações)
3. [Objetos](#objetos)
4. [Arrays](#arrays)
5. [Strings](#strings)
6. [Funções](#funções)
7. [Classes](#classes)
8. [Assincronismo](#assincronismo)
9. [Comparações](#comparações)
10. [Blocos e ponto e vírgula](#blocos-e-ponto-e-vírgula)
11. [Espaços em branco](#espaços-em-branco)
12. [Nomenclatura](#nomenclatura)
13. [Comentários](#comentários)
14. [DOM sem jQuery](#dom-sem-jquery)
15. [Testes](#testes)

## Módulos

Um arquivo, um módulo. Extensão `.js` (ou `.mjs` se o projeto exigir). `import`/`export` no topo. Sem CommonJS (`require`/`module.exports`) em código novo.

```javascript
import { formatMoney } from './money.js';
import { InvoiceApi } from '../api/invoice-api.js';

export function createInvoiceView(api) {
  return {
    async render(id) {
      const invoice = await api.get(id);
      return formatMoney(invoice.total);
    },
  };
}

export { InvoiceApi };
```

Prefira exports nomeados. `export default` só quando o módulo tem um único ponto de entrada óbvio (página ou componente raiz).

Importe com caminhos relativos explícitos e extensão `.js` quando o runtime ESM exigir.

## Tipos e declarações

Primitivos (`string`, `number`, `boolean`, `null`, `undefined`, `bigint`, `symbol`) passam por valor. Objetos, arrays e funções passam por referência.

`const` por padrão. `let` quando a variável precisa ser reatribuída. Nunca `var`.

```javascript
const taxRate = 0.1;
let attempt = 0;

attempt += 1;
```

Declare no menor escopo possível, perto do primeiro uso. Uma declaração por variável.

```javascript
const items = getItems();
const goSportsTeam = true;
```

## Objetos

Literal de objeto. Atalho de propriedade e método. Desestruturação na leitura.

```javascript
const name = 'Ada';
const user = {
  name,
  role: 'admin',
  greet() {
    return `Olá, ${this.name}`;
  },
};

const { role } = user;
```

Spread para copiar ou mesclar: `const next = { ...user, role: 'editor' };`.

Não use palavras reservadas como chave. Prefira `hidden` a `private`, `klass` só se a API externa forçar.

Encadeamento opcional e coalescência:

```javascript
const city = user.address?.city ?? 'Não informado';
```

## Arrays

Literal `[]`. `push`, `map`, `filter`, `find`, `includes`, `at`. Cópia com spread ou `Array.from`.

```javascript
const items = [];
items.push('abracadabra');

const copy = [...items];
const ids = users.map((user) => user.id);
```

Rest em parâmetros e desestruturação:

```javascript
const [first, ...rest] = ids;
```

## Strings

Aspas simples para strings simples. Template literal para interpolação e multilinha.

```javascript
const name = 'Bob Parr';
const fullName = `${name} ${this.lastName}`;

const errorMessage = `This is a super long error that was thrown because of Batman.
When you stop to think about how Batman had anything to do with this, you would get nowhere fast.`;
```

Não concatene com `+` só para quebrar linha. Não use `Array#join` para montar HTML; use template literal ou o DOM.

## Funções

Função nomeada para declarações reutilizáveis. Arrow function para callbacks curtos e para preservar `this` léxico.

```javascript
function parseQuery(search) {
  return new URLSearchParams(search);
}

const totals = items.map((item) => item.price * item.qty);
```

Parâmetros com padrão, rest e desestruturação:

```javascript
function connect({ host, port = 443, ...options } = {}) {
  return { host, port, options };
}
```

Não declare `function` dentro de `if`/`for`. Atribua uma expressão.

Não nomeie parâmetro de `arguments`. Use rest: `function log(level, ...messages) {}`.

## Classes

`PascalCase`. Campos de instância. Métodos no corpo da classe. Privados com `#` quando o encapsulamento for real.

```javascript
class User {
  #passwordHash;

  constructor({ name, passwordHash }) {
    this.name = name;
    this.#passwordHash = passwordHash;
  }

  hasName() {
    return Boolean(this.name);
  }
}

const user = new User({ name: 'Ada', passwordHash: '…' });
```

Getters booleanos: `isActive()`, `hasAge()`. Encadeamento retorna `this` só em APIs fluentes.

## Assincronismo

`async`/`await`. Trate erros com `try/catch` no limite do módulo (handler, controller, event listener).

```javascript
export async function loadInvoice(id) {
  const response = await fetch(`/api/invoices/${id}`);

  if (!response.ok) {
    throw new Error(`Falha ao carregar fatura ${id}`);
  }

  return response.json();
}
```

Paralelo com `Promise.all`. Sequência só quando uma chamada depende da anterior.

Não misture `.then()` com `await` no mesmo fluxo. Não esconda rejeição: todo `async` precisa de tratamento no chamador.

## Comparações

`===` e `!==`. Atalhos booleanos quando o valor já é o critério (`if (name)`, `if (items.length)`).

```javascript
if (status === 'open') {
  return true;
}

const total = Number(inputValue);
const hasAge = Boolean(age);
```

`parseInt` com base: `parseInt(inputValue, 10)`. Prefira `Number()` ou `BigInt()` quando o formato for conhecido.

## Blocos e ponto e vírgula

Sempre ponto e vírgula. Chaves em todo bloco, inclusive de uma linha.

```javascript
if (test) {
  return false;
}
```

Vírgula no fim do item, não no começo da linha.

## Espaços em branco

2 espaços por nível. Espaço antes de `{`. Linha em branco no fim do arquivo. Sem espaço antes de `(` em chamadas.

```javascript
function test() {
  console.log('test');
}

button.addEventListener('click', () => {
  panel.classList.toggle('is-open');
});
```

Encadeamento em várias linhas:

```javascript
const leds = stage
  .selectAll('.led')
  .data(data)
  .enter()
  .append('svg:svg');
```

## Nomenclatura

Descritivo. `camelCase` para funções, variáveis e instâncias. `PascalCase` para classes. `UPPER_SNAKE_CASE` para constantes de módulo.

```javascript
const MAX_RETRY_COUNT = 3;

function queryUsers() {}

const user = new User({ name: 'Bob Parr' });
```

Arquivos de módulo em `kebab-case.js`: `invoice-api.js`, `format-money.js`.

Prefixo `_` não cria privacidade. Use `#` em classes ou feche a variável no módulo.

Para o `this` léxico, prefira arrow function. Se precisar de alias, `const self = this` é aceitável; evite `_this` e `that`.

## Comentários

Comentários em português, acima da instrução. Docblock em funções públicas.

```javascript
/**
 * Converte centavos em texto monetário no locale pt-BR.
 *
 * @param {number} cents
 * @returns {string}
 */
export function formatMoney(cents) {
  return new Intl.NumberFormat('pt-BR', {
    style: 'currency',
    currency: 'BRL',
  }).format(cents / 100);
}
```

`// TODO:` trabalho pendente. `// FIXME:` defeito conhecido.

## DOM sem jQuery

APIs nativas: `document.querySelector`, `querySelectorAll`, `classList`, `closest`, `dataset`, `addEventListener`.

```javascript
const form = document.querySelector('#invoice-form');

form.addEventListener('submit', async (event) => {
  event.preventDefault();

  const data = new FormData(form);
  const payload = Object.fromEntries(data.entries());

  await fetch('/api/invoices', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(payload),
  });
});
```

Delegação:

```javascript
document.querySelector('#items').addEventListener('click', (event) => {
  const row = event.target.closest('[data-invoice-id]');

  if (!row) {
    return;
  }

  openInvoice(row.dataset.invoiceId);
});
```

Sem `$`, sem `.html()` com string de usuário, sem plugins jQuery.

## Testes

Cada módulo público tem teste. Nomes descrevem o comportamento: `formatMoney formata centavos em real brasileiro`.

Cubra o caminho feliz, entrada vazia e erro de rede. Veja [Padrões de testes](padroes-testes.md).
