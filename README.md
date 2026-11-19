<p align="center">
  <img src="https://diegopn.github.io/potoo-site/src/assets/images/language-interpreter-icon.svg?v=1c651ca7" alt="Potoo IDE language interpreter icon" width="120" />
</p>

<h1 align="center">Potoo Portugol Studio 2.0 Interpreter</h1>

<p align="center">An independent Rust interpreter library and command-line runner for Portugol Studio 2.0.</p>

<p align="center">
  <a href="https://github.com/diegopn/potoo-portugol-studio-2-interpreter/releases/latest">Download</a> ·
  <a href="https://diegopn.github.io/potoo-site/">Website</a> ·
  <a href="https://diegopn.github.io/potoo-site/manual/portugol-studio/">User manual</a>
</p>

<p align="center">
  <a href="https://www.rust-lang.org/"><img alt="Rust" src="https://img.shields.io/badge/Rust-000000?style=for-the-badge&logo=rust&logoColor=white"></a>
  <a href="https://doc.rust-lang.org/cargo/"><img alt="Cargo" src="https://img.shields.io/badge/Cargo-DEA584?style=for-the-badge&logo=rust&logoColor=white"></a>
  <img alt="Rust library" src="https://img.shields.io/badge/Rust_library-4A86CF?style=for-the-badge">
  <img alt="CLI" src="https://img.shields.io/badge/CLI-6A4C93?style=for-the-badge">
</p>

Originally developed as part of the Potoo IDE project, this interpreter is an independent Rust software package intended to be distributed separately from the IDE. It can be embedded as a library in other Rust projects or run as a standalone command-line tool on Linux, macOS, and Windows.

The crate depends on `indexmap` and can be embedded in Rust applications.

## Original language references

- [Portugol Studio official site (UNIVALI LITE)](https://univali-lite.github.io/Portugol-Studio/)
- [Upstream Portugol Studio source repository](https://github.com/UNIVALI-LITE/Portugol-Studio)

These upstream materials are the references for Portugol Studio 2.0 compatibility. This repository provides an independent Rust implementation.

## Original language references

- [Portugol Studio official site (UNIVALI LITE)](https://univali-lite.github.io/Portugol-Studio/)
- [Upstream Portugol Studio source repository](https://github.com/UNIVALI-LITE/Portugol-Studio)

These upstream materials are the references for Portugol Studio 2.0 compatibility. This repository provides an independent Rust implementation.

## Source example

```por
programa {
  funcao inicio() {
    escreva("Olá, Portugol!")
  }
}
```

## Use

Add the crate as a dependency:

```toml
[dependencies]
potoo-portugol-interpreter = { git = "https://github.com/diegopn/potoo-portugol-studio-2-interpreter.git", rev = "0e7e799b83daa80a3e8189327d5d7f9126ae69ef" }
```

Provide input and output through `RuntimeIo`, then call `execute`:

```rust
use potoo_portugol_interpreter::{RuntimeIo, execute};

struct Console;

impl RuntimeIo for Console {
    fn write(&mut self, text: &str) {
        print!("{text}");
    }

    fn read_line(&mut self) -> Result<String, String> {
        let mut input = String::new();
        std::io::stdin()
            .read_line(&mut input)
            .map_err(|error| error.to_string())?;
        Ok(input.trim_end_matches(['\r', '\n']).to_owned())
    }
}

fn main() -> Result<(), Vec<potoo_portugol_interpreter::Diagnostic>> {
    let source = r#"programa {
  funcao vazio inicio() {
    escreva("Olá, Portugol!")
  }
}"#;
    execute(source, &mut Console)
}
```

Call `parse` instead of `execute` when a host needs to inspect the public syntax tree without running the program.

`RuntimeIo::clear_output` can be implemented by hosts that want to handle `limpa()`; its default implementation does nothing. `execute_with_options` accepts limits for executed statements, output bytes, string values, total allocated array elements, and time spent waiting. The defaults are 1,000,000 steps, 1 MiB of output, 1 MiB per string, 100,000 array elements, and 30 seconds of waiting per run. Recursive calls are capped at 32 levels to protect the host stack, and the parser rejects statement or expression nesting beyond 128 levels.

## Supported language features

- `programa`, typed and `vazio` functions, parameters, return values, and global variables
- `inteiro`, `real`, `logico`, `caracter`, and `cadeia`
- Local variables, constants, vectors and matrices (including initializers and inferred vector sizes), and array parameters
- Value and reference parameters, expressions, assignments, and arithmetic, bitwise, comparison, and logical operators
- `se` / `senao`, `enquanto`, `faca` / `enquanto`, `para`, `escolha` / `caso`, `pare`, and `retorne`
- `escreva`, `leia`, and `limpa`
- `Matematica`: `PI`, `potencia`, `raiz`, `arredondar`, `logaritmo`, trigonometric functions, `valor_absoluto`, `maior_numero`, and `menor_numero`
- `Texto`: `numero_caracteres`, `caixa_alta`, `caixa_baixa`, `substituir`, `preencher_a_esquerda`, `obter_caracter`, `posicao_texto`, and `extrair_subtexto`
- `Tipos`: common conversions between strings, integers, reals, logical values, and characters
- `Util`: vector and matrix sizes, `sorteia`, `aguarde`, time since first library use, and the user's home directory
- `Calendario`: current date and time fields, plus full, short, and abbreviated weekday names
- `Arquivos`: open, read, write, list, filter, create, delete, replace text with regular expressions, and select files. Hosts provide the file operations through `RuntimeIo`.
- `Objetos`: in-memory objects, typed scalar and vector properties, JSON and XML input, JSON output, and object release
- `Internet` and `ServicosWeb`: HTTP requests and image downloads through host callbacks
- `Graficos`: every function and constant exported by the reference library is dispatched through `RuntimeIo::portugol_graphics`
- `Mouse` and `Teclado`: all exported functions and constants; input events and cursor control use `RuntimeIo` callbacks
- `Sons`: all exported functions through `RuntimeIo::portugol_audio`
- Library aliases such as `inclua biblioteca Matematica --> m`

## Dialect reference

This crate targets **Portugol Studio 2.0**. The [Potoo language reference](https://diegopn.github.io/potoo-site/en/manual/portugol-studio/) identifies the upstream sources used for compatibility work. There is no single language-wide Portugol standard that covers all tools and dialects.

The interpreter does not claim full compatibility yet: its parser reports the first lexical or syntax error, and graphics, keyboard, mouse, audio, HTTP, file, and calendar behavior depends on the host implementing the corresponding `RuntimeIo` callbacks. The Potoo IDE now provides GTK host handlers for graphics, keyboard, mouse, and audio, but those interactive features still need manual validation on each supported desktop platform. Runtime diagnostics include a source line and column.

## Development

Run these commands from the repository root:

```sh
cargo fmt --all -- --check
cargo check
cargo test
```

## Support

If this interpreter is useful in your work, you can support its continued development:

<p align="left">
  <a href="https://github.com/sponsors/diegopn"><img alt="Become a sponsor" src="https://img.shields.io/badge/Become_a_sponsor-EA4AAA?style=for-the-badge&logo=githubsponsors&logoColor=white"></a>
</p>

## Project policies

- [Code of conduct](CODE_OF_CONDUCT.md)
- [Contributing](CONTRIBUTING.md)
- [Security policy](SECURITY.md)
- [GPL-3.0-or-later license](LICENSE)

## Standalone command-line runner

Save this Portugol Studio 2.0 program as `example.por`:

```por
programa { funcao inicio() { escreva("Olá, Portugol!") } }
```

Run one source file without Potoo IDE:

```sh
potoo-portugol-studio-2 example.por
```

The example prints `Olá, Portugol!`. The terminal runner provides standard input/output. Host services for graphics, keyboard, mouse, audio, HTTP, project files, and calendar are unavailable; the interpreter reports those callbacks as unsupported. The parser reports the first lexical or syntax error, and the runner does not provide an interactive debugger.

## Install a prebuilt runner

The v0.1.1 release provides native CLI packages for Linux x86_64 GNU, Windows x86_64 MSVC, Intel macOS x86_64, and Apple silicon macOS arm64. Download your system's archive and `SHA256SUMS` from the [latest GitHub release](https://github.com/diegopn/potoo-portugol-studio-2-interpreter/releases/latest), then run the matching commands below from the folder containing both files. Each archive contains `potoo-portugol-studio-2` (or `potoo-portugol-studio-2.exe` on Windows) and the repository license.

### Linux x86_64 GNU

```sh
set -euo pipefail
archive="potoo-portugol-studio-2-interpreter-v0.1.1-x86_64-unknown-linux-gnu.tar.gz"
grep -F "  $archive" SHA256SUMS | sha256sum --check -
tar -xzf "$archive"
mkdir -p "$HOME/.local/bin"
install -m 755 "${archive%.tar.gz}/potoo-portugol-studio-2" "$HOME/.local/bin/potoo-portugol-studio-2"
export PATH="$HOME/.local/bin:$PATH"
potoo-portugol-studio-2 example.por
```

To keep the command available in new Linux sessions, add the `export PATH="$HOME/.local/bin:$PATH"` line to your shell profile.

### Windows x86_64 MSVC (PowerShell)

```powershell
$archive = "potoo-portugol-studio-2-interpreter-v0.1.1-x86_64-pc-windows-msvc.zip"
$line = Get-Content .\SHA256SUMS | Where-Object { $_.EndsWith("  $archive") } | Select-Object -First 1
if (-not $line) { throw "No SHA256SUMS entry found for $archive" }
$expected = ($line -split '\s+')[0]
$actual = (Get-FileHash -LiteralPath $archive -Algorithm SHA256).Hash.ToLowerInvariant()
if ($actual -ne $expected) { throw "SHA-256 mismatch for $archive" }
$installDir = Join-Path $env:USERPROFILE ".local\bin"
New-Item -ItemType Directory -Force $installDir | Out-Null
Expand-Archive -LiteralPath $archive -DestinationPath $installDir
$exe = Join-Path $installDir "potoo-portugol-studio-2-interpreter-v0.1.1-x86_64-pc-windows-msvc\potoo-portugol-studio-2.exe"
& $exe .\example.por
```

To run `potoo-portugol-studio-2.exe` from any folder, add `$env:USERPROFILE\.local\bin\potoo-portugol-studio-2-interpreter-v0.1.1-x86_64-pc-windows-msvc` to your user `Path` environment variable in Windows Settings.

### Intel macOS x86_64

```sh
set -euo pipefail
archive="potoo-portugol-studio-2-interpreter-v0.1.1-x86_64-apple-darwin.tar.gz"
grep -F "  $archive" SHA256SUMS | shasum -a 256 -c -
tar -xzf "$archive"
mkdir -p "$HOME/.local/bin"
install -m 755 "${archive%.tar.gz}/potoo-portugol-studio-2" "$HOME/.local/bin/potoo-portugol-studio-2"
export PATH="$HOME/.local/bin:$PATH"
potoo-portugol-studio-2 example.por
```

### Apple silicon macOS arm64

```sh
set -euo pipefail
archive="potoo-portugol-studio-2-interpreter-v0.1.1-aarch64-apple-darwin.tar.gz"
grep -F "  $archive" SHA256SUMS | shasum -a 256 -c -
tar -xzf "$archive"
mkdir -p "$HOME/.local/bin"
install -m 755 "${archive%.tar.gz}/potoo-portugol-studio-2" "$HOME/.local/bin/potoo-portugol-studio-2"
export PATH="$HOME/.local/bin:$PATH"
potoo-portugol-studio-2 example.por
```

On macOS, add the `export PATH="$HOME/.local/bin:$PATH"` line to `~/.zprofile` to keep the command available in future sessions.


## Install from source

With Rust 1.93 or later and network access, Cargo can install the command directly from its tagged source:

```sh
cargo install --git https://github.com/diegopn/potoo-portugol-studio-2-interpreter.git --tag v0.1.1 --bin potoo-portugol-studio-2
```

## Use as a Rust library

```toml
[dependencies]
potoo-portugol-interpreter = { git = "https://github.com/diegopn/potoo-portugol-studio-2-interpreter.git", rev = "0e7e799b83daa80a3e8189327d5d7f9126ae69ef" }
```

The library API lets Rust applications provide their own `RuntimeIo` implementation.

## DOI and citation

DOI: [10.6084/m9.figshare.34914840](https://doi.org/10.6084/m9.figshare.34914840)

Suggested reference (ABNT NBR 6023:2018):

> NASCIMENTO, Diego Pereira do. *Portugol Studio 2.0 interpreter v0.1.0*. 2026. Programa de computador. Disponível em: https://doi.org/10.6084/m9.figshare.34914840. Acesso em: 4 nov. 2026.
