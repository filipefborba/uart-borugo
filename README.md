# uart-borugo

Documentação

Explicar a comunicação UART
Exibir a forma de onda gerada pela implementação (usando o analog discovery)
Explicar o código


## Protocolo de comunicação UART

<img src='./UART.png'/>

Basicamente a comunicação UART tem 4 elementos principais, o start bit, data bits, parity bits,
e o stopbit. O startbit representa um sinal que informa o recebedor de que os bits que virão em seguida ja são a informação que quer ser enviada (os data bits). Para checar a validade dos dados recebidos temos o parity bit que informa se a quantidade de bits High recebidos são par ou impar, com essa informação o recebedor consegue passar pelos databits e fazer esse check. Para declarar o final do sinalenviado têm-se o stopbit, que são 2 bits em high. Por padrão o protocolo tem que o databits são 8 bits, e opcionalmente pode haver um 9 bit que é o bit de paridade.

## Forma de onda gerada pela implementação 



## Explicação do código 

A estrutura do código foi divida em duas pastas ,uma contendo os arquivos responsáveis pela recepção e outra
os arquivos da transmissão. Tanto para transmissão quanto para recepção, há três arquivos.

# Os .h sao responsáveis por:
    Responsável por importar as bibliotecas utilizadas no projeto.

# Os .cpp sao responsáveis por:

# Os .ino sao responsáveis por:
    É a camada mais alta da conexão, seu papel basicamente é chamar a lógica que está em mais baixo nível nos arquivos C++;
