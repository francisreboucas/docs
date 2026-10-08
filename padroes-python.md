# Padrões Python

Este guia descreve o estilo Python da equipe. A base normativa é [PEP 8](https://peps.python.org/pep-0008/), [PEP 484](https://peps.python.org/pep-0484/) (type hints) e [PEP 257](https://peps.python.org/pep-0257/) (docstrings). O formatador do projeto é [Ruff](https://docs.astral.sh/ruff/) (`ruff format` + `ruff check`).

Nenhum recurso novo entra no repositório sem testes que passem.

## Dicas rápidas

* Python 3.11+ salvo o projeto documentar outra versão. Sem Python 2.
* 4 espaços. Sem tabs. Linha-alvo 88 caracteres (padrão Ruff/Black).
* `snake_case` em funções, variáveis e módulos. `PascalCase` em classes. `UPPER_SNAKE_CASE` em constantes.
* Type hints em APIs públicas. `from __future__ import annotations` quando o projeto ainda precisar.
* Virtualenv ou equivalente. Dependências em `pyproject.toml`.
* `is` só com `None`, `True`, `False` singletons. Compare com `==` no restante.

## Ferramentas

```toml
[project]
name = "app"
requires-python = ">=3.11"

[tool.ruff]
line-length = 88
target-version = "py311"

[tool.ruff.lint]
select = ["E", "W", "F", "I", "B", "UP", "N", "S", "C4"]
ignore = ["E501"]
```

```text
ruff format .
ruff check .
pyright   # ou mypy
pytest
```

CI falha se o formatador alterar o diff. Ruff não substitui o type checker.

## Módulos e pacotes

Pacotes em minúsculas, sem underscore quando o nome for uma palavra (`billing`). Módulos `snake_case.py`. Evite `from module import *`.

```python
from datetime import datetime
from decimal import Decimal

from app.clock import Clock
from app.invoice.errors import InvoiceAlreadyIssued
from app.invoice.models import Invoice, Status
```

Ordem (Ruff I): stdlib, terceiros, locais. Uma linha em branco entre grupos.

`__all__` só em pacotes que publicam API explícita.

## Nomenclatura

| Elemento | Forma | Exemplo |
|---|---|---|
| Pacote | minúsculas | `invoice` |
| Módulo | snake_case | `invoice_service.py` |
| Classe / exceção | PascalCase | `InvoiceService`, `InvoiceAlreadyIssuedError` |
| Função / método / variável | snake_case | `issue_invoice`, `total_cents` |
| Constante | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| Interno | prefixo `_` | `_parse_row` |
| TypeVar | PascalCase curto | `T`, `RepoT` |

Exceções terminam em `Error` quando representam erro. Booleanos: `is_open`, `has_items`.

Não use `l`, `O`, `I` sozinhos. Dunder (`__init__`, `__str__`) só para o protocolo Python.

## Type hints

```python
from dataclasses import dataclass
from datetime import datetime
from enum import StrEnum


class Status(StrEnum):
    OPEN = "open"
    PAID = "paid"


@dataclass(frozen=True, slots=True)
class Invoice:
    id: str
    status: Status
    issued_at: datetime | None = None


def issue(invoice: Invoice, clock: Clock) -> Invoice:
    if invoice.status is not Status.OPEN:
        raise InvoiceAlreadyIssuedError(invoice.id)
    return Invoice(id=invoice.id, status=Status.PAID, issued_at=clock.now())
```

Use `X | Y` e `X | None` (PEP 604). `Optional[X]` só em código legado. `list[str]`, `dict[str, int]`, `tuple[int, ...]`.

`None` padrão anotado:

```python
def find(email: str, include_deleted: bool = False) -> Invoice | None:
    ...
```

Não use mutável como default (`def f(items=[])`). Use `None` e crie a lista no corpo.

## Funções e classes

Duas linhas em branco entre definições de topo. Uma linha em branco entre métodos.

```python
class InvoiceService:
    def __init__(self, repo: InvoiceRepository, clock: Clock) -> None:
        self._repo = repo
        self._clock = clock

    def issue(self, invoice_id: str) -> Invoice:
        invoice = self._repo.get(invoice_id)
        issued = issue(invoice, self._clock)
        self._repo.save(issued)
        return issued
```

Dataclasses para dados. Classe com comportamento quando há invariante ou dependências. `@property` para derivado barato; método para ação.

## Espaços e comparações

Espaço em volta de `=`, `==`, `->`. Sem espaço em default sem anotação de tipo no estilo PEP 8 clássico; com anotação, espaços em volta do `=`.

```python
def connect(host: str, port: int = 443) -> None:
    if host is None:
        raise ValueError("host is required")
```

`if status == Status.OPEN`. `if items:` em coleções. `if name is None:`.

## Docstrings e comentários

Docstrings em inglês só se o projeto publicar API internacional; neste repositório, **docstrings e comentários em português**. Módulo, classe e função públicos têm docstring.

```python
def format_money(cents: int) -> str:
    """Converte centavos em texto monetário no locale pt-BR."""
    return f"R$ {cents / 100:.2f}"
```

Comentário `#` explica decisão. `# TODO:` e `# FIXME:` iguais aos outros guias.

## Async e I/O

`async def` com um único loop (`asyncio`). Não chame função bloqueante dentro de coroutine sem `asyncio.to_thread`. HTTP: cliente async (`httpx.AsyncClient`) em serviços async.

```python
async def load_invoice(invoice_id: str, client: httpx.AsyncClient) -> Invoice:
    response = await client.get(f"/invoices/{invoice_id}")
    response.raise_for_status()
    return parse_invoice(response.json())
```

## Segurança

Sem `eval`, `exec` ou `pickle` de dado externo. SQL com parâmetros (`%s`, `:name`), nunca f-string de query. Segredos em ambiente. Ruff `S` (bandit) ligado. Veja [Padrões de segurança](padroes-seguranca.md).

## Testes

`pytest`. Arquivos `test_*.py`. Nomes em português descritivo:

```python
def test_issue_recusa_fatura_ja_emitida(clock: Clock) -> None:
    invoice = Invoice(id="inv_01", status=Status.PAID)

    with pytest.raises(InvoiceAlreadyIssuedError):
        issue(invoice, clock)
```

Fixtures em `conftest.py`. Sem rede pública. Veja [Padrões de testes](padroes-testes.md).
