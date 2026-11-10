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

The crate depends on `indexmap` and can be embedded in Rust applications.

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

## Project policies

- [Code of conduct](CODE_OF_CONDUCT.md)
- [Contributing](CONTRIBUTING.md)
- [Security policy](SECURITY.md)
- [GPL-3.0-or-later license](LICENSE)

## Standalone command-line runner

The `potoo-portugol-studio-2` command runs one source file without Potoo IDE. For example:

```sh
potoo-portugol-studio-2 example.por
```

The terminal runner provides standard input/output. Host services for graphics, keyboard, mouse, audio, HTTP, project files, and calendar are unavailable; the interpreter reports those callbacks as unsupported. The runner does not provide an interactive debugger.

## Install a prebuilt runner

Prebuilt binaries are published on the [GitHub Releases page](https://github.com/diegopn/potoo-portugol-studio-2-interpreter/releases). Choose a version and the package for your system there. Packages are currently available for Linux x86_64, Windows x86_64 (MSVC), and Intel macOS (x86_64). The current release does not include an Apple silicon (arm64) package.

Download only the package for your system and the `SHA256SUMS` file, and save both in the same folder. Before extracting or running the executable, compare the package's SHA-256 hash with its corresponding entry in `SHA256SUMS`. Run the commands below from that folder. Replace `example.por` with the path to a source file you want to run; the release package does not include a sample file.

### Linux x86_64

```sh
asset=$(ls -1 potoo-portugol-studio-2-interpreter-v*-x86_64-unknown-linux-gnu.tar.gz | head -n 1)
sha256sum "$asset" # compare with its line in SHA256SUMS
tar -xzf "$asset"
mkdir -p "$HOME/.local/bin"
install -m 755 "${asset%.tar.gz}/potoo-portugol-studio-2" "$HOME/.local/bin/potoo-portugol-studio-2"
export PATH="$HOME/.local/bin:$PATH"
potoo-portugol-studio-2 example.por
```

To keep the command available in new Linux sessions, add the `export PATH="$HOME/.local/bin:$PATH"` line to your shell profile.

### Windows x86_64 (PowerShell)

```powershell
$archive = Get-ChildItem -Filter "potoo-portugol-studio-2-interpreter-v*-x86_64-pc-windows-msvc.zip" | Select-Object -First 1
Get-FileHash $archive.FullName -Algorithm SHA256 # compare with the matching SHA256SUMS entry
Expand-Archive $archive.FullName -DestinationPath .\potoo-portugol-studio-2
$exe = Get-ChildItem .\potoo-portugol-studio-2 -Filter "potoo-portugol-studio-2.exe" -Recurse | Select-Object -First 1
& $exe.FullName .\example.por
```

To run `potoo-portugol-studio-2.exe` from another folder, add the directory containing the executable to your user `PATH`.

### Intel macOS

The published macOS package targets Intel Macs (x86_64). There is no prebuilt Apple silicon package in the current release.

```sh
asset=$(ls -1 potoo-portugol-studio-2-interpreter-v*-x86_64-apple-darwin.tar.gz | head -n 1)
shasum -a 256 "$asset" # compare with its line in SHA256SUMS
tar -xzf "$asset"
mkdir -p "$HOME/.local/bin"
install -m 755 "${asset%.tar.gz}/potoo-portugol-studio-2" "$HOME/.local/bin/potoo-portugol-studio-2"
export PATH="$HOME/.local/bin:$PATH"
potoo-portugol-studio-2 example.por
```

Add the `export PATH="$HOME/.local/bin:$PATH"` line to `~/.zprofile` to keep the command available in future macOS sessions.


## Install from source

With Rust 1.93 or later and network access, Cargo can install the command directly from its tagged source:

```sh
cargo install --git https://github.com/diegopn/potoo-portugol-studio-2-interpreter.git --rev 0e7e799b83daa80a3e8189327d5d7f9126ae69ef --bin potoo-portugol-studio-2
```

## Use as a Rust library

```toml
[dependencies]
potoo-portugol-interpreter = { git = "https://github.com/diegopn/potoo-portugol-studio-2-interpreter.git", rev = "0e7e799b83daa80a3e8189327d5d7f9126ae69ef" }
```

The library API lets Rust applications provide their own `RuntimeIo` implementation.

## Support

If this interpreter is useful in your work, you can support its continued development:

<p align="left">
  <a href="https://github.com/sponsors/diegopn"><img alt="Become a sponsor" src="https://img.shields.io/badge/Become_a_sponsor-EA4AAA?style=for-the-badge&logo=githubsponsors&logoColor=white"></a>
</p>

## DOI and citation

DOI: [10.6084/m9.figshare.34914840](https://doi.org/10.6084/m9.figshare.34914840)

Suggested reference (ABNT NBR 6023:2018):

> NASCIMENTO, Diego Pereira do. *Portugol Studio 2.0 interpreter v0.1.0*. 2026. Programa de computador. Disponível em: https://doi.org/10.6084/m9.figshare.34914840. Acesso em: 4 nov. 2026.
