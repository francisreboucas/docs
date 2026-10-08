# Padrões HTML

HTML semântico, acessível e previsível. O documento descreve estrutura; CSS descreve aparência; JavaScript descreve comportamento.

## Documento

Declare o doctype, o idioma e o charset. Viewport em páginas web.

```html
<!DOCTYPE html>
<html lang="pt-BR">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Faturas — Acme</title>
    <link rel="stylesheet" href="/css/app.css">
  </head>
  <body>
    <header>
      <p><a href="#conteudo">Saltar para o conteúdo</a></p>
      <nav aria-label="Principal">…</nav>
    </header>
    <main id="conteudo">…</main>
    <footer>…</footer>
    <script type="module" src="/js/app.js"></script>
  </body>
</html>
```

Indentação com 2 espaços. Atributos em minúsculas. Aspas duplas nos valores. Scripts de comportamento no fim do `body` com `type="module"`.

## Semântica

Use o elemento que expressa o significado:

| Uso | Elemento |
|---|---|
| Região principal | `main` (um por página) |
| Navegação | `nav` |
| Agrupamento autônomo | `article`, `section` |
| Lista | `ul`, `ol`, `dl` |
| Tabela de dados | `table` com `th` e `caption` |
| Botão de ação | `button` |
| Navegação para URL | `a href` |
| Texto enfatizado | `strong`, `em` — não `b`/`i` por estilo |

Não use `div` ou `span` quando existe elemento com significado. Não use tabela para layout.

## Formulários

Todo controle visível tem `label` associado (`for` + `id`). Agrupe com `fieldset` e `legend` quando fizer sentido. Indique obrigatório no rótulo e com `required`.

```html
<form action="/invoices" method="post" novalidate>
  <input type="hidden" name="_token" value="…">

  <div>
    <label for="customer-email">E-mail do cliente</label>
    <input
      id="customer-email"
      name="email"
      type="email"
      autocomplete="email"
      required
    >
  </div>

  <button type="submit">Salvar</button>
</form>
```

`button` sempre com `type` explícito (`submit`, `button` ou `reset`). Mensagens de erro ligadas ao campo com `aria-describedby`.

## Imagens e mídia

`img` tem `alt` descritivo. `alt=""` quando a imagem é só decoração. Informe `width` e `height` para reduzir salto de layout.

```html
<img src="/img/invoice-empty.svg" width="240" height="160" alt="">
<img src="/img/ada.png" width="96" height="96" alt="Retrato de Ada Lovelace">
```

Vídeo e áudio têm controles nativos e alternativa em texto.

## Acessibilidade

* Contraste e foco visível ficam no CSS; o HTML precisa permitir tabulação.
* Ícones sem texto visível têm nome acessível: `aria-label` ou texto em `.visually-hidden`.
* Títulos em ordem (`h1`–`h6`) sem pular nível por estética.
* `aria-*` só quando o HTML nativo não chega.
* Interações de teclado: Enter/Espaço em botões, Escape em diálogos.

```html
<button type="button" aria-expanded="false" aria-controls="menu-conta">
  Conta
</button>
```

## Atributos de dados

Estado e ganchos de JavaScript em `data-*`, nomes em kebab-case.

```html
<tr data-invoice-id="128" data-status="open">
```

Não use `id` como seletor de comportamento em massa. `id` é âncora única na página.

## Comentários e templates

Comentários em português, curtos. Não deixe markup comentado no repositório. Fragmentos reutilizáveis ficam em templates do backend ou em módulos JS; não copie HTML morto nas páginas.
