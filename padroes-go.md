# Padrões Go

Este guia descreve o estilo Go da equipe. A base normativa é [`gofmt`](https://pkg.go.dev/cmd/gofmt), [Effective Go](https://go.dev/doc/effective_go) e o [Google Go Style Guide](https://google.github.io/styleguide/go/). O formato do `gofmt` prevalece sobre preferência pessoal.

Nenhum recurso novo entra no repositório sem testes que passem.

## Dicas rápidas

* Rode `gofmt` (ou `goimports`) em todo arquivo. Indentação é tab, definida pela ferramenta.
* Pacote em uma palavra, minúsculas, sem underscore: `billing`, `httputil`.
* Exportado começa com maiúscula (`Issue`). Não exportado começa com minúscula (`issue`).
* Nomes em `MixedCaps` / `mixedCaps`. Siglas inteiras no mesmo caso: `HTTP`, `url`, `userID`.
* Erro é valor. Trate todo `error`. Não use `_` para descartar erro.
* `context.Context` é o primeiro parâmetro de I/O e RPC.
* Módulo com `go.mod`. Imports agrupados por `goimports`.

## Ferramentas

```text
gofmt -w .
goimports -w .
go test ./...
go vet ./...
```

CI falha se `gofmt` ou `goimports` alterarem o diff. `golangci-lint` pode complementar o `vet`.

## Pacotes e arquivos

Um diretório, um pacote. Nome do pacote igual ao último segmento do import, curto e no singular de domínio (`time`, `bytes`, `invoice`). Evite `util`, `common`, `helpers` como pacote raiz.

```go
package invoice

import (
	"context"
	"errors"

	"github.com/example/app/internal/clock"
)
```

Ordem dos imports: padrão, linha em branco, externos, linha em branco, internos. `goimports` faz isso.

Arquivo `doc.go` só quando o pacote precisa de visão geral. Comentário de pacote começa com `Package invoice ...`.

## Nomenclatura

| Elemento | Forma | Exemplo |
|---|---|---|
| Pacote | uma palavra, minúsculas | `invoice` |
| Tipo exportado | PascalCase | `InvoiceService` |
| Função / método | PascalCase ou camelCase | `Issue`, `issueNow` |
| Constante | MixedCaps | `MaxRetryCount` |
| Receiver | 1–2 letras, estável | `func (s *Service)` |
| Interface de um método | verbo + `er` | `Reader`, `Issuer` |
| Arquivo | snake_case.go | `invoice_service.go` |
| Teste | `_test.go` | `invoice_service_test.go` |

Não repita o nome do pacote no identificador: `invoice.Service`, não `invoice.InvoiceService`, quando o pacote já diz o domínio.

```go
owner := obj.Owner()
if owner != user {
	obj.SetOwner(user)
}
```

Getters não usam prefixo `Get`: `Owner()`, não `GetOwner()`. Booleanos: `IsOpen()`, `HasItems()`.

## Erros

Crie erros com `fmt.Errorf` e `%w` para encadear. Compare com `errors.Is` / `errors.As`.

```go
var ErrAlreadyIssued = errors.New("invoice: already issued")

func (s *Service) Issue(ctx context.Context, id string) (*Invoice, error) {
	inv, err := s.repo.Get(ctx, id)
	if err != nil {
		return nil, fmt.Errorf("issue invoice %s: %w", id, err)
	}

	if inv.Status != StatusOpen {
		return nil, ErrAlreadyIssued
	}

	inv.IssuedAt = s.clock.Now()
	if err := s.repo.Save(ctx, inv); err != nil {
		return nil, fmt.Errorf("save invoice %s: %w", id, err)
	}

	return inv, nil
}
```

Mensagem de erro em minúsculas, sem ponto final, em inglês (contrato de log e comparação). Comentários de código em português.

Não pânico em biblioteca. `panic` só em invariante realmente impossível no `main` ou em `Must*` documentado.

## Context

Primeiro parâmetro: `ctx context.Context`. Nunca guarde `Context` em struct, salvo o que o ciclo de vida da struct possui de ponta a ponta (e mesmo assim prefira passar na chamada). Respeite `ctx.Done()`.

```go
func (r *Repository) Get(ctx context.Context, id string) (*Invoice, error) {
	row := r.db.QueryRowContext(ctx, `SELECT id, status FROM invoices WHERE id = $1`, id)
	// ...
}
```

## Concorrência

Toda goroutine tem plano de término (contexto, `WaitGroup` ou canal). Não crie goroutine sem quem a espere ou cancele. Canais para sinal e pipeline; mutex para estado compartilhado simples.

```go
g, ctx := errgroup.WithContext(ctx)
g.Go(func() error {
	return s.notify(ctx, inv)
})
return g.Wait()
```

## Interfaces

Defina interfaces no pacote consumidor, pequenas. Aceite interfaces, devolva structs.

```go
type Clock interface {
	Now() time.Time
}
```

## Comentários e documentação

Comentário de identificador exportado é frase completa que começa com o nome:

```go
// Issue emite a fatura aberta e persiste o novo estado.
func (s *Service) Issue(ctx context.Context, id string) (*Invoice, error) {
	// Relógio injetado para o teste controlar o instante da emissão.
	inv.IssuedAt = s.clock.Now()
	return inv, nil
}
```

`TODO(nome):` e `FIXME:` com dono quando o item tiver responsável.

## Testes

Pacote `_test` externo quando o teste deve ver só a API pública. Table-driven tests:

```go
func TestIssueRecusaFaturaJaEmitida(t *testing.T) {
	t.Parallel()

	tests := []struct {
		name   string
		status Status
		want   error
	}{
		{name: "paga", status: StatusPaid, want: ErrAlreadyIssued},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			t.Parallel()
			// ...
		})
	}
}
```

Sem rede pública. Relógio e I/O injetados. Veja [Padrões de testes](padroes-testes.md).

## Exemplo completo

```go
package invoice

import (
	"context"
	"errors"
	"fmt"
	"time"
)

type Status string

const (
	StatusOpen Status = "open"
	StatusPaid Status = "paid"
)

type Invoice struct {
	ID       string
	Status   Status
	IssuedAt time.Time
}

type Repository interface {
	Get(ctx context.Context, id string) (*Invoice, error)
	Save(ctx context.Context, inv *Invoice) error
}

type Clock interface {
	Now() time.Time
}

type Service struct {
	repo  Repository
	clock Clock
}

func NewService(repo Repository, clock Clock) *Service {
	return &Service{repo: repo, clock: clock}
}

func (s *Service) Issue(ctx context.Context, id string) (*Invoice, error) {
	inv, err := s.repo.Get(ctx, id)
	if err != nil {
		return nil, fmt.Errorf("issue invoice %s: %w", id, err)
	}
	if inv.Status != StatusOpen {
		return nil, errors.New("invoice: already issued")
	}
	inv.IssuedAt = s.clock.Now()
	if err := s.repo.Save(ctx, inv); err != nil {
		return nil, fmt.Errorf("save invoice %s: %w", id, err)
	}
	return inv, nil
}
```
