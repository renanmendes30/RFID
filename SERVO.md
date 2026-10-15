# Tutorial: Controlando um servo motor com Arduino

**Material didático em Markdown --- versão resumida e reescrita**

## 1. O que é necessário

Para os primeiros testes:

-   Arduino UNO compatível;
-   Micro servo 9 g, por exemplo SG90;
-   Jumpers macho-macho;
-   Cabo USB para programar a placa.

Para as atividades adicionais:

-   Protoboard;
-   Potenciômetro linear de 10 kΩ (um ou dois, conforme a atividade);
-   Motor Shield L293D, caso vá realizar o exemplo com shield.

## 2. Identificação dos fios do servo

Em muitos micro servos SG90, os fios são:

| Fio típico | Função |
|---|---|
| Vermelho | Alimentação positiva, normalmente 5 V |
| Marrom ou preto | GND |
| Laranja ou amarelo | Sinal de controle |

As cores podem variar entre fabricantes. Confirme a identificação na
documentação do componente.

## 3. Cuidados com a alimentação

-   Conecte o GND do servo ao GND do circuito.
-   O servo pode exigir picos de corrente superiores aos que a placa
    consegue fornecer de forma estável. Se houver tremores,
    reinicializações ou desconexões, utilize uma fonte externa regulada
    e dimensionada para o servo.
-   Ao usar uma fonte externa, una o GND da fonte ao GND do Arduino.
-   Não conecte a saída positiva de uma fonte externa ao pino de 5 V do
    Arduino sem verificar a arquitetura e as recomendações da placa.
-   Não force o eixo do servo contra um batente mecânico.

## 4. Atividade 1 --- Movimento básico do servo

### Ligações

  | Servo | Arduino UNO |
|---|---|
| Sinal (laranja/amarelo) | D6 |
| Alimentação (vermelho) | 5 V apropriado |
| GND (marrom/preto) | GND |

### Código

``` cpp
#include <Servo.h>

Servo meuServo;

void setup() {
  meuServo.attach(6);  // Sinal conectado ao pino D6
  meuServo.write(0);   // Posição inicial
}

void loop() {
  // Move gradualmente de 0 a 90 graus
  for (int angulo = 0; angulo <= 90; angulo++) {
    meuServo.write(angulo);
    delay(15);
  }

  delay(1000);

  // Retorna gradualmente a 0 grau
  for (int angulo = 90; angulo >= 0; angulo--) {
    meuServo.write(angulo);
    delay(15);
  }

  delay(1000);
}
```

### O que observar

O servo deve mover-se entre as posições comandadas, com uma pausa nas
extremidades. Os valores de ângulo representam posições solicitadas, não
necessariamente uma medição exata do ângulo físico.

## 5. Atividade 2 --- Controle de posição com potenciômetro

O potenciômetro fornece uma entrada analógica que pode ser convertida em
uma posição do servo.

### Ligações do potenciômetro

  | Terminal do potenciômetro | Arduino UNO |
|---|---|
| Terminal externo 1 | GND |
| Terminal central | A0 |
| Terminal externo 2 | 5 V |

Mantenha as ligações do servo da atividade anterior.

### Código

``` cpp
#include <Servo.h>

Servo meuServo;

const int pinoPotenciometro = A0;
int leitura;
int angulo;

void setup() {
  meuServo.attach(6);
}

void loop() {
  leitura = analogRead(pinoPotenciometro);  // Faixa típica: 0 a 1023
  angulo = map(leitura, 0, 1023, 0, 180);
  meuServo.write(angulo);
  delay(15);
}
```

### O que observar

Ao girar o potenciômetro, o servo deve acompanhar a mudança de posição.
Se o servo não alcançar toda a faixa, reduza os limites de ângulo para
evitar pressionar os batentes mecânicos.



## Fonte de referência

MakerHero. [Controlando um servo motor com Arduino: 6 modos
diferentes](https://www.makerhero.com/blog/controlando-um-servo-motor-com-arduino/).
