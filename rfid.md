# Tutorial: Controle de acesso com RFID e Arduino

## 1. Objetivo

Montar um controle de acesso simples com Arduino UNO e leitor RFID
MFRC522. O Arduino lê o identificador único (UID) do cartão ou chaveiro
e compara esse valor com uma lista de identificadores autorizados. O
resultado pode ser exibido no Monitor Serial e em um display LCD.

## 2. Componentes

-   Arduino UNO
-   Módulo RFID MFRC522
-   Cartão ou chaveiro RFID compatível com o MFRC522
-   Display LCD 16×2 compatível com HD44780
-   Protoboard e jumpers
-   Potenciômetro de 10 kΩ para ajustar o contraste do LCD, caso o
    display use interface paralela
-   Cabos USB para programação

**Atenção:** o MFRC522 deve ser alimentado com 3,3 V. Não o alimente
diretamente com 5 V.

## 3. Ligações do leitor RFID

Para o Arduino UNO, o tutorial utiliza a interface SPI:

 | Pino do MFRC522 | Pino do Arduino UNO |
|---|---|
| SDA / SS | D10 |
| SCK | D13 |
| MOSI | D11 |
| MISO | D12 |
| RST | D9 |
| GND | GND |
| 3.3V | 3.3V |
| NC | Sem conexão |

## 4. Ligações do display LCD 16×2 em modo paralelo

<img width="2736" height="1143" alt="image" src="https://github.com/user-attachments/assets/fbba8788-39b7-4d43-9055-87c7b4113143" />


O tutorial utiliza um LCD com interface paralela, ligado à biblioteca
`LiquidCrystal`:

  | Pino/ligação do LCD | Pino do Arduino UNO |
|---|---|
| RS | D6 |
| E | D7 |
| D4 | D5 |
| D5 | D4 |
| D6 | D3 |
| D7 | D2 |
| RW | GND |
| VSS | GND |
| VDD | 5 V |
| VO | Pino central do potenciômetro de 10 kΩ |

Ligue os terminais externos do potenciômetro a 5 V e GND para ajustar o
contraste. Consulte a pinagem do seu modelo de LCD antes de energizar.

**Importante:** um LCD 2002 I²C é diferente do LCD paralelo acima. Se
estiver usando o LCD 2002 I²C disponível no laboratório, a ligação e a
biblioteca serão diferentes; normalmente são usados SDA e SCL, e o
código precisa ser adaptado.

## 5. Preparar a IDE Arduino

1.  Abra a Arduino IDE.
2.  Entre em **Ferramentas → Gerenciar Bibliotecas**.
3.  Pesquise e instale a biblioteca **MFRC522** (comunidade GitHub).
4.  A biblioteca `SPI` já acompanha a IDE Arduino.
5.  Para o LCD paralelo, utilize a biblioteca `LiquidCrystal`, também
    incluída na IDE.

## 6. Descobrir o UID dos cartões

Antes de cadastrar cartões autorizados:

1.  Carregue um exemplo de leitura de UID da biblioteca MFRC522 ou
    execute um programa que imprima o UID no Monitor Serial.
2.  Configure o Monitor Serial para a mesma velocidade definida no
    programa (por exemplo, 9600 baud).
3.  Aproxime um cartão ou chaveiro do leitor.
4.  Anote o UID hexadecimal apresentado.
5.  Repita para cada cartão que deseja testar.

O UID varia conforme o cartão. Não copie os identificadores de exemplos
da internet: utilize os valores lidos nos cartões reais do laboratório.



## 7. Lógica do programa

## Código Arduino — Controle de Acesso RFID

```cpp
//Programa : RFID - Controle de Acesso Modulo RFID com Arduino
//Autor : MakerHero

#include <SPI.h>
#include <MFRC522.h>
#include <LiquidCrystal.h>

#define SS_PIN 10
#define RST_PIN 9

MFRC522 mfrc522(SS_PIN, RST_PIN);

LiquidCrystal lcd(6, 7, 5, 4, 3, 2);

char st[20];

void setup()
{
  Serial.begin(9600);
  SPI.begin();
  mfrc522.PCD_Init();

  Serial.println("Aproxime o seu cartao do leitor...");
  Serial.println();

  // Define o número de colunas e linhas do LCD
  lcd.begin(16, 2);

  mensageminicial();
}

void loop()
{
  // Verifica se há um novo cartão
  if (!mfrc522.PICC_IsNewCardPresent())
  {
    return;
  }

  // Seleciona o cartão
  if (!mfrc522.PICC_ReadCardSerial())
  {
    return;
  }

  // Mostra o UID na porta serial
  Serial.print("UID da tag :");

  String conteudo = "";
  byte letra;

  for (byte i = 0; i < mfrc522.uid.size; i++)
  {
    Serial.print(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " ");
    Serial.print(mfrc522.uid.uidByte[i], HEX);

    conteudo.concat(
      String(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " ")
    );

    conteudo.concat(String(mfrc522.uid.uidByte[i], HEX));
  }

  Serial.println();
  Serial.print("Mensagem : ");

  conteudo.toUpperCase();

  // UID 1 - Chaveiro autorizado
  if (conteudo.substring(1) == "ED 78 03 CA")
  {
    Serial.println("Ola MakerHero !");
    Serial.println();

    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Ola MakerHero !");
    lcd.setCursor(0, 1);
    lcd.print("Acesso liberado!");

    delay(3000);
    mensageminicial();
  }

  // UID 2 - Cartão com acesso negado
  if (conteudo.substring(1) == "BD 9B 06 7D")
  {
    Serial.println("Ola Cartao !");
    Serial.println();

    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Ola Cartao !");
    lcd.setCursor(0, 1);
    lcd.print("Acesso Negado !");

    delay(3000);
    mensageminicial();
  }
}

void mensageminicial()
{
  lcd.clear();
  lcd.print(" Aproxime o seu");
  lcd.setCursor(0, 1);
  lcd.print("cartao do leitor");
}
```

O programa deve executar esta sequência:

1.  Inicializar a comunicação serial, o barramento SPI, o leitor RFID e
    o LCD.
2.  Mostrar uma mensagem solicitando a aproximação de um cartão.
3.  Detectar a presença de uma nova tag e ler seu UID.
4.  Converter o UID para um formato hexadecimal consistente.
5.  Comparar o UID com os identificadores cadastrados.
6.  Se houver correspondência, exibir **Acesso liberado**; caso
    contrário, exibir **Acesso negado**.
7.  Após alguns segundos, retornar à mensagem inicial.

### Exemplo de estrutura para cadastrar UIDs

Substitua os valores abaixo pelos UIDs que você realmente leu no Monitor
Serial:

``` cpp
// Exemplo ilustrativo: substitua pelos UIDs dos seus cartões.
const char* uidAutorizado = "AA BB CC DD";

// Depois de ler e formatar o UID do cartão em uma String chamada uidLido:
if (uidLido == uidAutorizado) {
  // Exibir "Acesso liberado"
} else {
  // Exibir "Acesso negado"
}
```

Esse trecho ilustra apenas a comparação. Ele não é um programa completo:
a leitura do MFRC522, a formatação do UID e o controle do LCD precisam
estar implementados no sketch.

## 8. Testes

-   Aproxime um cartão cadastrado e confirme a indicação de acesso
    autorizado.
-   Aproxime um cartão não cadastrado e confirme que o acesso seja
    negado.
-   Verifique se o UID aparece corretamente no Monitor Serial.
-   Teste cada cartão mais de uma vez.
-   Confira a alimentação e as ligações caso o leitor não detecte as
    tags.

## 9. Segurança e extensões

-   O UID de cartões RFID comuns não deve ser considerado uma credencial
    de alta segurança, pois pode ser copiado ou emulado.
-   Não utilize o protótipo como único mecanismo de segurança para uma
    porta real.
-   O projeto pode ser ampliado com servo motor para simular uma
    cancela, LEDs de status, buzzer e comunicação serial com ESP32.
-   Para integrar o ESP32, envie uma mensagem estruturada com o
    resultado da tentativa e valide os dados recebidos antes de
    utilizá-los.
-   Se o ESP32 publicar eventos via MQTT, use autenticação e TLS quando
    disponíveis.

## Fonte

MakerHero --- [Controle de Acesso usando Módulo RFID com
Arduino](https://www.makerhero.com/blog/controle-acesso-leitor-rfid-arduino/)
