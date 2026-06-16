# Requisitos

- Precisa receber a função pushMessage como parâmetro (função que gerencia o envio de mensagens na fila Bluetooth)
- A comunicação bluetooth é implementada pelo usuário da biblioteca. A interface entre o bluetooth e o código da biblioteca é feita através da função pushMessage.

# Features

- Exporta métodos para montar mensagens seguindo o protocolo de comunicação (wire protocol).
  - Enviar mensagens do tipo single.
  - Enviar mensagens do tipo lista. Montar o body desejado e automaticamente adicionar o header correspondente para cada mensagem.
  - Responder a requisições do tipo single.
  - Responder a requisições do tipo lista.
  - Auxilio no parse de números de ponto flutuante.
  - Auxilio no parse de wire strings para structs facilmente acessíveis como variáveis no código (melhor UX).
- Ter uma estrutura map de comandos para facilitar cadastrar novos comandos sem precisar modificar o código da biblioteca.
- Função para cadastrar novos comandos no map de comandos.
- Recebe o tamanho da mensagem (em bytes) como template parameter (para assign em tempo de compilação)

# Protocolo de comunicação (wire protocol)

- Comandos seguem a sintaxe de chamada de função em C++.

O tráfego com o dispositivo segue **comandos em texto** no estilo abaixo (não JSON). Uma **mensagem** pode conter **vários comandos** separados por **`;`** (o separador não conta dentro de parênteses nem dentro de strings entre aspas).

Caso uma lista muito longa seja enviada, a lista pode ser dividida em várias mensagens, que por sua vez são separadas em comandos por **`;`**. Cada mensagem do tipo lista é composta por um comando do tipo header e um ou mais comandos do tipo body.

Cada comando tem a forma:

```text
nome(modo, req_ou_resp, ...argumentos)
```

| Posição após `nome(`         | Significado       | Valores                                                                                            |
| ---------------------------- | ----------------- | -------------------------------------------------------------------------------------------------- |
| 1º parâmetro (`modo`)        | Tipo de comando   | **`s`** = *single* (comando único), **`h`** = *header* de lista, **`b`** = *body* (linha da lista) |
| 2º parâmetro (`req_ou_resp`) | Papel da mensagem | **`r`** = requisição, **`s`** = resposta                                                           |

Exemplos:

```text
help(s,r);
param_list(h,s,5,1,1,0);
param_list(b,s,1,"param_get","ref","read a parameter");
map_get(b,s,0,1,2,3,4,5);
```

- **Single** (`s`): após `r` ou `s` vêm os argumentos do comando, se houver.
- **Lista** (`h` / `b`): no **`h`**, após `r`/`s` vêm **quatro inteiros** `T,C,B,j` (total de linhas, linhas nesta mensagem, total de mensagens, índice da mensagem); ver `WireListHeader` em `api/protocol_utils.py` e `docs/wire_protocol_firmware_implementation.md`. No **`b`**, após `r`/`s` vem o **índice** da linha e os argumentos dessa linha.
- O firmware costuma limitar o tamanho de cada mensagem (ex.: **256 bytes** no NUS); listas longas são fatiadas em várias mensagens.