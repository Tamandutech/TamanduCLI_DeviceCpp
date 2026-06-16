# Exemplo de gerenciamento de parâmetros

Variáveis atômicas utilizadas como parâmetros de configuração do dispositivo. Essas variáveis são escritas no dispositivo através do protocolo de comunicação (wire protocol) e lidas pelo firmware em tempos diferentes para evitar race conditions.

## Fluxo `param_list`

O host envia uma **single request** sem argumentos:

```text
param_list(s,r);
```

O dispositivo responde com uma **lista** (`h` + um ou mais `b`), possivelmente dividida em várias mensagens BLE. Cada linha editável é um body com **índice**, **nome** e **valor**:

```text
param_list(h,s,3,3,1,0);
param_list(b,s,1,State.runOnMappingMode,1);
param_list(b,s,2,Vacuum.speed,75);
param_list(b,s,3,PID.kP,0.42);
```

| Campo no body | Significado                                     |
| ------------- | ----------------------------------------------- |
| índice        | Posição da linha (começa em **1**)              |
| nome          | Identificador do parâmetro (ex.: `Classe.nome`) |
| valor         | Valor atual como string                         |

O TamanduCLI coleta essas respostas, grava em `output/param_list.txt` e permite editar em `input/param_list.txt`. Mudanças são aplicadas com `param_set` por linha.

## Implementação no firmware

```cpp
#include "cli_map.hpp"
#include <atomic>
#include <cstdio>
#include <vector>

// Parâmetros atômicos — lidos/escritos em contextos diferentes sem mutex.
std::atomic<int>   g_runOnMappingMode{1};
std::atomic<int>   g_vacuumSpeed{75};
std::atomic<float> g_pidKp{0.42f};

static void pushMessage(const char *msg) {
  // Enfileira a string UTF-8 para envio via BLE NUS (use sua própria implementação do bluetooth).
  bleNusEnqueue(msg);
}

static void appendParamRow(std::vector<wire::Protocol<256>::ListRow> &rows,
                           int index, const char *name, const char *value) {
  char idx[16];
  snprintf(idx, sizeof(idx), "%d", index);
  rows.push_back({idx, name, value});
}

static bool onParamList(const wire::Command &cmd, wire::WireView view,
                        wire::Protocol<256> &proto) {
  (void)view;
  if (cmd.mode != 's' || cmd.role != 'r') {
    return false;
  }

  char buf[64];

  std::vector<wire::Protocol<256>::ListRow> rows;
  snprintf(buf, sizeof(buf), "%d", g_runOnMappingMode.load());
  appendParamRow(rows, 1, "State.runOnMappingMode", buf);

  snprintf(buf, sizeof(buf), "%d", g_vacuumSpeed.load());
  appendParamRow(rows, 2, "Vacuum.speed", buf);

  snprintf(buf, sizeof(buf), "%.4g", static_cast<double>(g_pidKp.load()));
  appendParamRow(rows, 3, "PID.kP", buf);

  return proto.emitListResponseRows("param_list", rows);
}

void setupCli() {
  static cli::CliMap<256> cli(pushMessage);
  cli.registerCommand("param_list", onParamList);
}

void onBleRx(const char *message) {
  static cli::CliMap<256> cli(pushMessage);
  cli.processMessage(message);  // ex.: "param_list(s,r);"
}
```

`emitListResponseRows` monta cada `param_list(b,s,…);`, adiciona o header `param_list(h,s,T,C,B,j);` em cada mensagem e chama `pushMessage` automaticamente quando a lista não cabe em um único pacote.

## Resposta esperada pelo host

Para os três parâmetros acima, o host recebe algo equivalente a:

```text
param_list(h,s,3,3,1,0);param_list(b,s,1,State.runOnMappingMode,1);param_list(b,s,2,Vacuum.speed,75);param_list(b,s,3,PID.kP,0.42);
```

Com muitos parâmetros, a mesma lista é fatiada em várias mensagens — cada uma com seu próprio header `h` e `B`/`j` atualizados.
