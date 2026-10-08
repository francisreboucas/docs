# Padrões Rust

Este guia descreve o estilo Rust da equipe. A base normativa é [`rustfmt`](https://github.com/rust-lang/rustfmt), [`clippy`](https://github.com/rust-lang/rust-clippy) e as [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/). O formato do `rustfmt` prevalece.

Nenhum recurso novo entra no repositório sem testes que passem.

## Dicas rápidas

* Rode `cargo fmt` e `cargo clippy --all-targets -- -D warnings` no CI.
* Edição 2021 (ou a edition do crate) em `Cargo.toml`. Sem `unsafe` sem comentário `SAFETY:` e revisão.
* Tipos em `PascalCase`. Funções, métodos e módulos em `snake_case`. Constantes e `static` em `SCREAMING_SNAKE_CASE`.
* Prefira `Result<T, E>` e `Option<T>` a pânico em biblioteca. `unwrap` só em testes ou invariante local óbvia.
* Erros com `thiserror` em bibliotecas e `anyhow` em binários, salvo o crate já ter política própria.
* Imports agrupados: std, crates externos, crate local.

## Ferramentas

```text
cargo fmt
cargo clippy --all-targets -- -D warnings
cargo test
cargo doc --no-deps
```

`rust-toolchain.toml` fixa o canal quando o projeto precisar de versão específica.

## Crates e módulos

Um crate, um `Cargo.toml`. Módulos em arquivo `snake_case.rs` ou diretório `snake_case/mod.rs`. `pub` só no que é contrato. Prefira `pub(crate)` para API interna.

```rust
mod repository;
mod service;

pub use service::InvoiceService;
```

Binário fino em `src/main.rs`; lógica em `src/lib.rs` para os testes unitários enxergarem a API.

## Nomenclatura

| Elemento | Forma | Exemplo |
|---|---|---|
| Crate / módulo | snake_case | `invoice_service` |
| Struct, enum, trait | PascalCase | `InvoiceService`, `Status` |
| Função / método / variável | snake_case | `issue_invoice` |
| Constante / static | SCREAMING_SNAKE_CASE | `MAX_RETRY_COUNT` |
| Lifetime | curto, minúsculo | `'a`, `'repo` |
| Tipo genérico | `PascalCase` curto | `T`, `E`, `Repo` |
| Trait conversor | verbo | `From`, `TryFrom` |
| Feature flag | snake_case | `serde` |

Siglas: `HttpClient`, `user_id`. Traits de comportamento: `Issuer`, `Clock`.

## Tipos e visibilidade

Campos privados por padrão. Construtores (`new`, `try_new`) validam. Enums para estados, não booleanos paralelos.

```rust
#[derive(Clone, Debug, Eq, PartialEq)]
pub enum Status {
    Open,
    Paid,
}

#[derive(Clone, Debug)]
pub struct Invoice {
    id: String,
    status: Status,
}

impl Invoice {
    pub fn id(&self) -> &str {
        &self.id
    }

    pub fn status(&self) -> Status {
        self.status
    }
}
```

Derive `Debug`, `Clone`, `Eq`/`PartialEq` quando o tipo for dado. `Copy` só em tipos realmente baratos.

## Erros e option

```rust
use thiserror::Error;

#[derive(Debug, Error)]
pub enum InvoiceError {
    #[error("invoice {0} already issued")]
    AlreadyIssued(String),
    #[error(transparent)]
    Storage(#[from] sqlx::Error),
}

pub fn issue(invoice: &mut Invoice) -> Result<(), InvoiceError> {
    if invoice.status() != Status::Open {
        return Err(InvoiceError::AlreadyIssued(invoice.id().to_owned()));
    }
    invoice.mark_issued();
    Ok(())
}
```

`?` para propagar. `expect("motivo")` quando o pânico é invariante. Evite `unwrap` em caminhos de produção.

`match` exaustivo. `if let` / `let else` para um ramo só.

```rust
let Some(user) = user else {
    return Err(InvoiceError::AlreadyIssued(id));
};
```

## Ownership e empréstimo

Aceite `&str` e `&[T]` em parâmetros quando o callee não precisa de posse. Devolva `String` / `Vec<T>` quando o caller ganha posse. Evite `.clone()` para calar o compilador; extraia a vida útil ou reestruture.

```rust
pub fn find_by_email(email: &str) -> Option<User> {
    // ...
}
```

`Arc<T>` para estado compartilhado entre tasks. `Mutex` / `RwLock` com seção crítica curta. Em async, `tokio::sync` em vez de `std::sync` quando o lock cruza `.await`.

## Async

Runtime explícito (`tokio`) no binário. Bibliotecas async sem `tokio::main`. Traits async com a edition e o compilador do projeto (RPITIT / `async fn` em trait quando estável no toolchain).

```rust
pub async fn issue(&self, id: &str) -> Result<Invoice, InvoiceError> {
    let mut invoice = self.repo.get(id).await?;
    invoice.mark_issued();
    self.repo.save(&invoice).await?;
    Ok(invoice)
}
```

Não bloqueie o runtime com `std::thread::sleep` ou CPU pesada sem `spawn_blocking`.

## unsafe

Bloco `unsafe` mínimo, encapsulado, com comentário `SAFETY:` listando as invariantes. Clippy `undocumented_unsafe_blocks` ligado.

```rust
// SAFETY: ptr veio de Vec::as_mut_ptr e len == cap do mesmo buffer.
unsafe {
    ptr::write(ptr.add(len), value);
}
```

## Documentação

`///` em itens públicos, em português. Exemplos compiláveis quando a API for biblioteca.

```rust
/// Emite a fatura aberta e persiste o novo estado.
///
/// # Errors
///
/// Retorna [`InvoiceError::AlreadyIssued`] se o status não for `Open`.
pub async fn issue(&self, id: &str) -> Result<Invoice, InvoiceError> {
    todo!()
}
```

`//` explica decisão. `todo!()`, `unimplemented!()` não passam no CI de produção.

## Testes

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn issue_recusa_fatura_ja_emitida() {
        let mut invoice = Invoice::open("inv_01");
        invoice.mark_issued();
        let err = issue(&mut invoice).unwrap_err();
        assert!(matches!(err, InvoiceError::AlreadyIssued(_)));
    }
}
```

Testes de integração em `tests/`. Fixtures em `tests/fixtures`. Veja [Padrões de testes](padroes-testes.md).

## Cargo.toml

Dependências com versão compatível (`1.2`). Features opt-in. `[dev-dependencies]` para teste. Sem `path` de máquina local no crate publicado.
