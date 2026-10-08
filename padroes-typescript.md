# Padrões TypeScript

Este guia descreve o estilo TypeScript da equipe. Complementa [Padrões JavaScript](padroes-javascript.md): ESM, `const`/`let`, sem jQuery, sem `var`. A base normativa é o [TSConfig strict](https://www.typescriptlang.org/tsconfig/#strict) e o guia JS deste repositório.

Nenhum recurso novo entra no repositório sem testes que passem.

## Dicas rápidas

* `"strict": true` no `tsconfig.json`. Sem `any`. Prefira `unknown` e estreite.
* Arquivos `.ts` / `.tsx`. ESM (`"module": "NodeNext"` ou `ESNext` conforme o runtime).
* 2 espaços, aspas simples, ponto e vírgula — iguais ao JavaScript.
* Tipos em `PascalCase`. Valores em `camelCase`. Enums e uniões de string em vez de `enum` numérico.
* `import type` para tipos só usados em posição de tipo.
* Não commite `.js` gerado quando o build reproduz. Versionar `*.d.ts` só em pacotes publicados.

## Ferramentas

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true,
    "skipLibCheck": true,
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "target": "ES2022"
  }
}
```

ESLint com `typescript-eslint` + o formatter do projeto (Prettier, se o repo já o usa). CI roda `tsc --noEmit` e os testes.

## Relação com JavaScript

Tudo em [padroes-javascript.md](padroes-javascript.md) vale: módulos nomeados, `async`/`await`, DOM nativo, nomenclatura de arquivos `kebab-case.ts`. Este documento cobre só o que o tipo acrescenta.

## Tipos

Anote contratos públicos (exports, props, handlers, DTO). Deixe o compilador inferir locais óbvios.

```typescript
export type InvoiceStatus = 'open' | 'paid';

export interface Invoice {
  id: string;
  status: InvoiceStatus;
  totalCents: number;
  issuedAt: Date | null;
}

export function isOpen(invoice: Invoice): boolean {
  return invoice.status === 'open';
}
```

`type` para uniões, interseções e aliases. `interface` para objeto extensível (props de componente, adapter). Não misture os dois para o mesmo conceito.

`readonly` em dados que o callee não muta. Tuplas só com significado posicional claro.

```typescript
function totals(values: readonly number[]): number {
  return values.reduce((sum, value) => sum + value, 0);
}
```

### `any`, `unknown`, asserts

```typescript
function parseInvoice(payload: unknown): Invoice {
  if (!isInvoice(payload)) {
    throw new Error('Payload de fatura inválido');
  }
  return payload;
}

function isInvoice(value: unknown): value is Invoice {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'status' in value
  );
}
```

`as` só na borda (DOM, JSON, biblioteca sem tipos). `as const` para literais.

Evite `!` (non-null assertion). Prefira estreitar ou lançar.

## Funções e genéricos

Genérico nomeado quando aparece mais de uma vez ou documenta o contrato. Constraints explícitas.

```typescript
export async function getJson<T>(url: string, parse: (data: unknown) => T): Promise<T> {
  const response = await fetch(url);

  if (!response.ok) {
    throw new Error(`Falha HTTP ${response.status}`);
  }

  return parse(await response.json());
}
```

Parâmetros opcionais no fim. Não use `foo?: T | undefined` à toa: `exactOptionalPropertyTypes` distingue ausência de `undefined`.

## Classes e módulos

Classes para identidade com ciclo de vida. Funções e tipos para dados e casos de uso.

```typescript
export class InvoiceApi {
  constructor(private readonly baseUrl: string) {}

  async get(id: string): Promise<Invoice> {
    return getJson(`${this.baseUrl}/invoices/${id}`, parseInvoice);
  }
}
```

Campos `readonly` no construtor. Visibilidade `private` / `protected` / `public` explícita em membros de classe. `#` privado só quando o encapsulamento precisa existir em runtime.

## Null e undefined

`undefined` = ausente. `null` = valor vazio deliberado na API (JSON, banco). Não misture os dois no mesmo campo.

```typescript
type Customer = {
  email: string;
  phone?: string;
  deletedAt: Date | null;
};
```

Encadeamento opcional e coalescência iguais ao JS: `user.address?.city ?? 'Não informado'`.

## React / JSX (quando o projeto usar)

Arquivo `.tsx`. Componentes em `PascalCase`. Props tipadas; não use `React.FC` só para ter `children`.

```tsx
type InvoiceCardProps = {
  invoice: Invoice;
  onIssue: (id: string) => void;
};

export function InvoiceCard({ invoice, onIssue }: InvoiceCardProps) {
  return (
    <article>
      <h2>{invoice.id}</h2>
      <button type="button" onClick={() => onIssue(invoice.id)}>
        Emitir
      </button>
    </article>
  );
}
```

Hooks: `use` + substantivo (`useInvoice`). Não chame hooks sob condição.

## Comentários

Comentários em português. JSDoc em exports quando o tipo sozinho não conta o *porquê*.

```typescript
/**
 * Converte centavos em texto monetário no locale pt-BR.
 */
export function formatMoney(cents: number): string {
  return new Intl.NumberFormat('pt-BR', {
    style: 'currency',
    currency: 'BRL',
  }).format(cents / 100);
}
```

Não repita o tipo no JSDoc (`@param {string}`) — o compilador já é a fonte.

## Testes

`*.test.ts` / `*.spec.ts`. Teste o comportamento com os tipos do domínio, não a inferência. `as Invoice` no teste só em fixture já válida. Veja [Padrões de testes](padroes-testes.md).
