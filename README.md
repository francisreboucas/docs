# Padrões de desenvolvimento fullstack

Guias de estilo para projetos PHP + JavaScript. O PHP segue [PSR-1](https://www.php-fig.org/psr/psr-1/), [PSR-4](https://www.php-fig.org/psr/psr-4/), [PSR-12](https://www.php-fig.org/psr/psr-12/) e [PER Coding Style](https://www.php-fig.org/per/coding-style/). O JavaScript segue ES2020+ com módulos nativos.

Use estes documentos em qualquer projeto da equipe. Ferramentas (`php-cs-fixer`, `phpcs`, ESLint, Prettier) devem refletir as mesmas regras.

## Documentos

* [Padrões PHP](padroes-php.md)
* [Padrões JavaScript](padroes-javascript.md)
* [Padrões HTML](padroes-html.md)
* [Padrões CSS](padroes-css.md)
* [Padrões SQL](padroes-sql.md)
* [Padrões HTTP e API](padroes-http-api.md)
* [Padrões Git](padroes-git.md)
* [Padrões de testes](padroes-testes.md)
* [Padrões de segurança](padroes-seguranca.md)

## Regras comuns

* Arquivos em UTF-8 sem BOM e quebra de linha LF.
* Documentação nestes guias em português.
* Identificadores de código (classes, funções, variáveis, tabelas) em inglês.
* Comentários de código em português.
* Uma responsabilidade por arquivo quando o idioma exigir (uma classe PHP por arquivo).
* Nenhum recurso novo entra no repositório sem testes que passem.
* Segredos, senhas e chaves ficam fora do código e do Git.

## Idioma e exemplos

Comentários explicam *por que*, não *o que* o código já diz. URLs e e-mails de exemplo usam `example.com`, `example.org` ou `example.net` ([RFC 2606](https://www.rfc-editor.org/rfc/rfc2606)).
