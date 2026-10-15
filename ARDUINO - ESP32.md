# Comunicação serial entre Arduino UNO e ESP32

A comunicação serial entre o Arduino UNO e o ESP32 é feita por UART, usando os pinos TX e RX. Os dispositivos podem enviar e receber dados, por exemplo, para o Arduino transmitir a leitura de um sensor e o ESP32 encaminhá-la para uma aplicação IoT.

## 1. Ligação dos fios

Considerando o **Arduino UNO clássico de 5 V** e um ESP32 convencional:

| Arduino UNO | ESP32 | Função |
|---|---|---|
| Pino 11 (TX) | GPIO 16 (RX), por divisor de tensão | Arduino envia dados |
| Pino 10 (RX) | GPIO 17 (TX) | Arduino recebe dados |
| GND | GND | Referência elétrica comum |

### Atenção à tensão

O TX do Arduino UNO clássico pode enviar 5 V, enquanto o ESP32 trabalha com lógica de 3,3 V. **Não conecte diretamente o TX de 5 V do UNO ao RX do ESP32.** Use um divisor de tensão ou conversor de nível lógico nessa direção.

Um divisor simples pode usar:
- Resistor de **1 kΩ** entre TX do UNO e RX do ESP32.
- Resistor de **2 kΩ** entre RX do ESP32 e GND.

Confira as ligações do divisor antes de energizar o circuito.

## 2. Código do Arduino UNO

Neste exemplo, o Arduino lê um potenciômetro conectado ao pino A0 e envia o valor ao ESP32. A biblioteca `SoftwareSerial` usa os pinos 10 e 11, deixando a USB livre para programação e depuração.

```cpp
#include <SoftwareSerial.h>

// RX = 10, TX = 11
SoftwareSerial comunicacao(10, 11);

void setup() {
  Serial.begin(9600);
  comunicacao.begin(9600);

  pinMode(A0, INPUT);
}

void loop() {
  int valor = analogRead(A0);

  comunicacao.println(valor);

  Serial.print("Enviado ao ESP32: ");
  Serial.println(valor);

  delay(1000);
}
```

## 3. Código do ESP32

O ESP32 recebe os dados pela UART2 e exibe a leitura no Monitor Serial do computador.

```cpp
#define RX_ESP32 16
#define TX_ESP32 17

void setup() {
  Serial.begin(115200);

  // UART2: RX, TX
  Serial2.begin(9600, SERIAL_8N1,
                RX_ESP32, TX_ESP32);

  Serial.println("Aguardando dados do Arduino...");
}

void loop() {
  if (Serial2.available()) {
    String mensagem = Serial2.readStringUntil('\n');
    mensagem.trim();

    if (mensagem.length() > 0) {
      Serial.print("Recebido do Arduino: ");
      Serial.println(mensagem);
    }
  }
}
```

## 4. Ligação do potenciômetro

- Terminal externo 1: **GND** do Arduino.
- Terminal central: **A0** do Arduino.
- Terminal externo 2: **5 V** do Arduino.

Ao girar o potenciômetro, o Arduino deverá enviar valores de aproximadamente **0 a 1023**.

## 5. Como testar

1. Carregue o código do Arduino UNO na placa.
2. Carregue o código do ESP32 na placa.
3. Abra o Monitor Serial do ESP32 em **115200 baud**.
4. Gire o potenciômetro.
5. Confira os valores recebidos no Monitor Serial.

A comunicação entre as placas usa **9600 baud**, enquanto o Monitor Serial do ESP32 usa **115200 baud**. Isso é normal: são duas conexões seriais distintas.

## 6. Resumo das conexões

| Sinal | Conexão |
|---|---|
| TX do Arduino (pino 11) | RX do ESP32 (GPIO 16), com divisor de tensão |
| RX do Arduino (pino 10) | TX do ESP32 (GPIO 17) |
| GND do Arduino | GND do ESP32 |
| Potenciômetro | 5 V, GND e A0 do Arduino |

## 7. Referência

- [Documentação oficial da Serial do ESP32 (Arduino)](https://docs.espressif.com/projects/arduino-esp32/en/latest/api/serial.html)

> **Observação:** este exemplo considera o Arduino UNO clássico e um ESP32 convencional. Outros modelos podem ter diferenças de pinos e de compatibilidade elétrica.
