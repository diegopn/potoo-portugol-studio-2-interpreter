# Potoo Portugol Studio 2.0 Interpreter

A reusable Portugol Studio 2.0 interpreter library written in Rust. The crate depends on `indexmap`, can be embedded in Rust applications, and includes a standalone terminal runner.

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

This crate targets **Portugol Studio 2.0**. The [Potoo language reference](https://diegopn.github.io/potoo-site/en/manual/portugol/) identifies the upstream sources used for compatibility work. There is no single language-wide Portugol standard that covers all tools and dialects.

The interpreter does not claim full compatibility yet: its parser reports the first lexical or syntax error, and graphics, keyboard, mouse, audio, HTTP, file, and calendar behavior depends on the host implementing the corresponding `RuntimeIo` callbacks. The Potoo IDE now provides GTK host handlers for graphics, keyboard, mouse, and audio, but those interactive features still need manual validation on each supported desktop platform. Runtime diagnostics include a source line and column.

## Development

Run these commands from the repository root:

```sh
cargo fmt --all -- --check
cargo check
cargo test
```

## Project policies

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

Download the archive for your system and `SHA256SUMS` from the [latest GitHub Release](https://github.com/diegopn/potoo-portugol-studio-2-interpreter/releases/latest). Binary releases are not published yet. The planned first release will be tagged `v0.1.0`; its release workflow is configured to produce these asset names:

| System | Archive |
| --- | --- |
| Linux x86_64 | `potoo-portugol-studio-2-interpreter-v0.1.0-x86_64-unknown-linux-gnu.tar.gz` |
| Windows x86_64 | `potoo-portugol-studio-2-interpreter-v0.1.0-x86_64-pc-windows-msvc.zip` |
| macOS Intel | `potoo-portugol-studio-2-interpreter-v0.1.0-x86_64-apple-darwin.tar.gz` |


Current CI and release targets are Linux x86_64, Windows x86_64, and macOS Intel (x86_64). Apple silicon (arm64) is not supported in this release; support may be added in a future version.

### Linux x86_64

```sh
asset=potoo-portugol-studio-2-interpreter-v0.1.0-x86_64-unknown-linux-gnu.tar.gz
release=https://github.com/diegopn/potoo-portugol-studio-2-interpreter/releases/download/v0.1.0
curl -fLO "$release/$asset"
curl -fLO "$release/SHA256SUMS"
sha256sum "$asset" # compare with its line in SHA256SUMS
tar -xzf "$asset"
mkdir -p "$HOME/.local/bin"
install -m 755 "potoo-portugol-studio-2-interpreter-v0.1.0-x86_64-unknown-linux-gnu/potoo-portugol-studio-2" "$HOME/.local/bin/potoo-portugol-studio-2"
potoo-portugol-studio-2 example.por
```

Add it for the current shell with `export PATH="$HOME/.local/bin:$PATH"`; place that line in your shell profile to keep it for future sessions.

### Windows x86_64 (PowerShell)

```powershell
$asset = "potoo-portugol-studio-2-interpreter-v0.1.0-x86_64-pc-windows-msvc.zip"
$release = "https://github.com/diegopn/potoo-portugol-studio-2-interpreter/releases/download/v0.1.0"
Invoke-WebRequest "$release/$asset" -OutFile $asset
Invoke-WebRequest "$release/SHA256SUMS" -OutFile SHA256SUMS
Get-FileHash ".\$asset" -Algorithm SHA256 # compare with the matching SHA256SUMS entry
Expand-Archive ".\$asset" -DestinationPath .\potoo-portugol-studio-2-interpreter
.\potoo-portugol-studio-2-interpreter\potoo-portugol-studio-2-interpreter-v0.1.0-x86_64-pc-windows-msvc\potoo-portugol-studio-2.exe .\example.por
```

To run `potoo-portugol-studio-2.exe` from any folder, add its extracted directory to your user `PATH` in Windows Environment Variables.

### macOS Intel

This release supports Intel Macs only. Apple silicon is not supported in this version.

```sh
asset=potoo-portugol-studio-2-interpreter-v0.1.0-x86_64-apple-darwin.tar.gz
release=https://github.com/diegopn/potoo-portugol-studio-2-interpreter/releases/download/v0.1.0
curl -fLO "$release/$asset"
curl -fLO "$release/SHA256SUMS"
shasum -a 256 "$asset" # compare with its line in SHA256SUMS
tar -xzf "$asset"
mkdir -p "$HOME/.local/bin"
install -m 755 "${asset%.tar.gz}/potoo-portugol-studio-2" "$HOME/.local/bin/potoo-portugol-studio-2"
potoo-portugol-studio-2 example.por
```

Add it for the current shell with `export PATH="$HOME/.local/bin:$PATH"`; place that line in `~/.zprofile` to keep it for future macOS sessions.

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

## DOI

A stable Figshare DOI will be added when the first public release is published.
