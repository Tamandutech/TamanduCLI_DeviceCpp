# TamanduCLI - Biblioteca de Dispositivo em C++

Biblioteca para integração com o TamanduCLI.

## Recursos

- Protocolo de comunicação (wire protocol) com o TamanduCLI.
- Mapeamento de strings de comando para chamadas de função.

## Primeiros passos

1. Clone o repositório

```bash
git clone https://github.com/platformio/esp32.pio.template.git
```

2. Instale o PlatformIO Core (CLI)

[PlatformIO Core](https://docs.platformio.org/en/latest/core/installation/index.html)

```bash
pip install platformio
```

3. Instale o PlatformIO IDE (VSCode com a extensão PlatformIO)

[PlatformIO IDE](https://docs.platformio.org/en/latest/integration/ide/vscode.html)

4. Instale as extensões recomendadas

Consulte o arquivo `.vscode/extensions.json` para ver as extensões recomendadas.

> Ao abrir o projeto no VSCode, as extensões a serem instaladas serão exibidas automaticamente.

5. Execute os comandos para instalar as dependências e compilar o projeto

```bash
pio install
pio run -e esp32 --target compiledb
```
