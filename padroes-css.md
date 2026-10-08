# Padrões CSS

CSS previsível, de baixa especificidade e independente de JavaScript de apresentação. Indentação com 2 espaços.

## Arquivos

Um arquivo global de tokens (cores, espaçamento, tipografia) e arquivos por domínio (`invoice.css`, `form.css`). Nome de arquivo em `kebab-case.css`.

Ordem sugerida no arquivo:

1. `@import` ou `@use` (se o preprocessador exigir)
2. Tokens / custom properties
3. Reset ou normalize do projeto
4. Layout
5. Componentes
6. Utilitários
7. Estados e media queries

## Nomenclatura (BEM)

Blocos em kebab-case. Elemento com `__`. Modificador com `--`.

```css
.invoice-card {
}

.invoice-card__title {
}

.invoice-card--overdue {
}

.invoice-card.is-open {
}
```

`is-*` e `has-*` para estado runtime ligado pelo HTML/JS. Sem IDs como seletor de estilo. Sem seletor de tag + classe (`div.invoice-card`) salvo reset.

## Indentação e formatação

Uma declaração por linha. Espaço depois dos dois-pontos. Ponto e vírgula em toda declaração. Chave de abertura na mesma linha do seletor.

```css
.invoice-card {
  display: flex;
  flex-direction: column;
  gap: var(--space-sm);
  padding: var(--space-md);
  color: var(--color-text);
  background-color: var(--color-surface);
}
```

## Custom properties

Tokens no `:root` (ou no escopo do tema). Nomes em kebab-case, prefixo por categoria.

```css
:root {
  --color-text: #1b1b1b;
  --color-surface: #ffffff;
  --space-sm: 0.5rem;
  --space-md: 1rem;
  --font-sans: "Source Sans 3", system-ui, sans-serif;
}
```

Cores e espaços no componente apontam para tokens, não para hex solto — salvo exceção documentada (sombra de terceiros, override pontual).

## Unidades

* Layout fluido: `%`, `fr`, `min()`, `max()`, `clamp()`.
* Tipografia e espaço: `rem`.
* Borda e sombra estáticas: `px`.
* Evite `em` em padding de componente (compõe de forma surpresa). Evite `px` em `font-size`.

## Especificidade

Prefira uma classe. Evite `!important`. Evite seletor de ID. Encadeie no máximo bloco + elemento + modificador.

```css
/* preferido */
.form-field__error {
  color: var(--color-danger);
}

/* evite */
#invoice-form div span.error {
  color: red !important;
}
```

## Media queries

Mobile first. Quebras em tokens ou valores explícitos do projeto.

```css
.invoice-grid {
  display: grid;
  gap: var(--space-md);
}

@media (min-width: 48rem) {
  .invoice-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}
```

## Foco e movimento

Todo controle interativo tem estilo de `:focus-visible`. Respeite `prefers-reduced-motion`.

```css
.button:focus-visible {
  outline: 2px solid var(--color-focus);
  outline-offset: 2px;
}

@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

## HTML e CSS

Classes de estilo no HTML; comportamento no JS. Não use estilo inline salvo valor realmente dinâmico (ex.: `style="width: 42%"` em barra de progresso calculada).
