# uart-borugo

# Protocolo de comunicação UART

<img src='./UART.png'/>

Basicamente a comunicação UART tem 4 elementos principais, o start bit, data bits, parity bits,
e o stopbit. O startbit representa um sinal que informa o recebedor de que os bits que virão em seguida ja são a informação que quer ser enviada (os data bits). Para checar a validade dos dados recebidos temos o parity bit que informa se a quantidade de bits High recebidos são par ou impar, com essa informação o recebedor consegue passar pelos databits e fazer esse check. Para declarar o final do sinalenviado têm-se o stopbit, que são 2 bits em high. Por padrão o protocolo tem que o databits são 8 bits, e opcionalmente pode haver um 9 bit que é o bit de paridade.

# Forma de onda gerada pela implementação 



# Explicação do código 

A estrutura do código foi divida em duas pastas, uma contendo os arquivos responsáveis pela recepção e outra os arquivos da transmissão. Tanto para transmissão quanto para recepção, há três arquivos.

#### Os .h sao responsáveis por:
    Importar as bibliotecas utilizadas no projeto.

#### Os .cpp sao responsáveis por:
    Lógica do programa; seria a camada enlace do primeiro projeto. É onde se encontram os métodos para envio e recepção.

#### Os .ino sao responsáveis por:
    É a camada mais alta da conexão, seu papel basicamente é chamar a lógica que está em mais baixo nível nos arquivos C++.

## TX - Transmissão

### .ino
Este é o código que é carregado no Arduíno. É responsável por importar a biblioteca e o arquivo de enlace, setar a velocidade do serial e efetivamente controlar a comunicação.

```ino
#include "sw_uart.h"

due_sw_uart uart;

void setup() {
  Serial.begin(9600);
  sw_uart_setup(&uart, 19, 18, 1, 8, SW_UART_EVEN_PARITY);
}

void loop() {
 test_write();
}

void test_write() {
  sw_uart_write_string(&uart,"Cam-Fisica\n");
  delay(50);
}
```

### .cpp

O seguinte código corresponde à lógica da comunicação UART. As funções realizam os processos necessários para que a comunicação seja estabelecida, enquanto o .ino apenas controla essas funções. A primeira função a ser descrita é a de cálculo de paridade ímpar: a função recebe o payload e realiza a contagem de cada bit e soma todos os bits 1. Ao final da leitura, se a soma não tem resto quando dividida por 2, a paridade está correta, então ela retorna que o payload está correto.

```cpp
int calc_even_parity(char data) {
  int count = 0;
  int a;

  for (int i = 0; i < 8; i++){
    a = (data >> i) & 0x01;
    count += a;
  }

  if (count % 2 == 0) {
    return 1;
  }

  else {
    return 0;
  }
}
```

A seguinte função é responsável por realizar o envio de uma string via UART. Mais explicações estão dentro do próprio código.

```cpp
//Função para enviar um char (data 8 bits) via UART
void sw_uart_write_byte(due_sw_uart *uart, char data) {
  
  //Variável para armazenar paridade
  int parity = 0;

  //Calcula a paridade e atualiza a variável acima, 1 indica que está certo, 0 indica que está errado
  if(uart->paritybit == SW_UART_EVEN_PARITY) {
     parity = calc_even_parity(data);
  } else if(uart->paritybit == SW_UART_ODD_PARITY) {
     parity = !calc_even_parity(data);
  }
  
  //Envia o start bit, que é 0
  digitalWrite(uart -> pin_tx, LOW);
  _sw_uart_wait_T(uart);
  
  //Realiza um for que envia cada bit do payload
  for(int i = 0; i < uart->databits; i++) {
    digitalWrite(uart -> pin_tx, (data >> i) & 0x01);
    _sw_uart_wait_T(uart);
  }

  //Envia paridade, se existir
  if(uart->paritybit != SW_UART_NO_PARITY) {
   digitalWrite(uart -> pin_tx, parity);
   _sw_uart_wait_T(uart);
  }
  
  //Envia stop bit, se existir
  for(int i = 0; i < uart->stopbits; i++) {
    digitalWrite(uart -> pin_tx, HIGH);
    _sw_uart_wait_T(uart);
  } 
}
```

## RX - Recepção
### .ino
Este é o código que é carregado no Arduíno. É responsável por importar a biblioteca e o arquivo de enlace, setar a velocidade do serial e efetivamente controlar a comunicação. Ao receber um sinal, verifica se há um erro na paridade. Se não tiver, printa a mensagem, caso contrário, printa "PARITY ERROR".

```ino
#include "sw_uart.h"

due_sw_uart uart;

void setup() {
  Serial.begin(9600);
  sw_uart_setup(&uart, 19, 18, 1, 8, SW_UART_EVEN_PARITY);
}

void loop() {
 test_receive();
}

void test_write() {
  sw_uart_write_string(&uart,"camFisica\n");
  delay(50);
}

void test_receive() {
  char data;
  int code = sw_uart_receive_byte(&uart, &data);
  if(code == SW_UART_SUCCESS) {
     Serial.print(data);
  } else if(code == SW_UART_ERROR_PARITY) {
    Serial.println("PARITY ERROR");
  } else {
    Serial.println("OTHER");
    Serial.print(code);
  }
}
```

### .cpp

Assim como no TX, o seguinte código corresponde à lógica da comunicação UART. A seguinte função é responsável por realizar a recepção de uma string via UART. Mais explicações estão dentro do próprio código.

```cpp
//Recebimento de dados do Serial
int sw_uart_receive_byte(due_sw_uart *uart, char* data) {

  //Variável para recebimento de dados
  char nchar  = 0;
  
  //Variável para calculo da paridade
  char parity, rx_parity;
  
  //Aguarda start bit
  bool startBitFound = false;
  while (!startBitFound) {
    //Confirma start BIT
    if (digitalRead(uart -> pin_rx) == 0) {
      _sw_uart_wait_half_T(uart);
      //Checa se bit ainda é 0
      if (digitalRead(uart -> pin_rx) == 0) {
        startBitFound = true;
      }
    }
  }

  _sw_uart_wait_T(uart);
  
  //Recebe dados
  for (int i = 0; i < 8; i++){
    nchar = nchar|(digitalRead(uart -> pin_rx) << i);
    _sw_uart_wait_T(uart);
  }

  //Recebe a paridade
  rx_parity = digitalRead(uart -> pin_rx);
  _sw_uart_wait_half_T(uart);

  //Recebe stop bit
  int stopBit = digitalRead(uart -> pin_rx);
  
  //Checa se a paridade está correta
  parity = calc_even_parity(nchar);

  if(parity != rx_parity) {
    return SW_UART_ERROR_PARITY;
  }
  
  *data = nchar;
  return SW_UART_SUCCESS;
}
```