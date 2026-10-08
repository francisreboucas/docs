# Padrões Git

Histórico linear o bastante para rastrear *o que* mudou e *por quê*. Commits pequenos. Mensagens em português, no infinitivo ou no indicativo, como o repositório já usa.

## Branches

| Branch | Papel |
|---|---|
| `main` ou `master` | Produção. Só entra via revisão. |
| `feature/descricao-curta` | Trabalho novo |
| `fix/descricao-curta` | Correção |
| `chore/descricao-curta` | Tooling, docs, dependências |

Nome em kebab-case, ASCII. Uma intenção por branch. Atualize com rebase na branch base enquanto o trabalho for local; merge da base se a branch já estiver compartilhada e o rebase for destrutivo.

## Commits

Um commit, uma intenção. Não misture formatação com regra de negócio.

Assunto com até ~72 caracteres, primeira letra maiúscula, sem ponto final. Corpo opcional: contexto, risco, como testar.

```
Adicionar emissão de fatura com evento de cobrança

A emissão só ocorre em faturas open. Testes em
InvoiceServiceTest cobrem o conflito 409.
```

Evite commits `WIP`, `ajustes` e `fix` sem objeto. Prefixe se o time usar escopo: `docs:`, `fix:`, `feat:` — de forma consistente.

Não commite:

* `.env`, chaves, dumps, `node_modules/`, `vendor/`
* arquivos gerados que o build reproduz
* código comentado “para depois”

## Pull requests

Título igual ao assunto do commit principal ou um resumo da branch. Descrição com:

* o problema
* a abordagem
* como testar
* riscos (migração, contrato de API, feature flag)

PR pequeno. Screenshot ou gravação só quando a mudança for visual.

Revise o próprio diff antes de pedir review. Atenda comentários com commits novos ou com um único commit de follow-up claro; evite `git commit --amend` e force-push depois que outra pessoa já revisou, salvo acordo explícito.

## Tags e releases

Tags semânticas: `v1.4.0`. Changelog por versão, agrupado em adições, mudanças, correções e quebras.

## .gitignore

Ignore artefatos de IDE, OS, build e segredos. Se um arquivo ignorado precisar ser versionado (ex.: `.env.example`), force só o exemplo, nunca o valor real.
