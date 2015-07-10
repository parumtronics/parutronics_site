---
title: "E quando não havia o Arduino?"
layout: blog_page
date: 2015-02-19
---

Quando comecei a brincar com microcontroladores, alguns anos atrás, havia somente os embriões de toda essa 
corrente maker que agora flui para alçar voos maiores e com um monte de projetos bacanas possibilitando a execução 
rápida e com cada vez mais recursos. Isso faz com que eu me sinta um tanto "old school". Mas não me tome por tão velho assim. 
Comecei a estudar/brincar com sistemas embarcados por volta de 2001, não é tanto tempo assim.

Desde então muita coisa mudou. Naqueles tempos o pessoal por aqui no Brasil praticamente só falava em PIC, do 
mesmo modo que hoje muita gente só conhece o Arduino. Bem o ponto deste post é justamente esse: existe um mundo
imenso fora dessas plataformas a ser explorado e é um mundo bem divertido. As plataformas como o Arduino são 
sensacionais dentro da sua proposta mas não são tudo o que há ou sempre houve. Então, e se não houvesse o Arduino no mundo,
como fazer?
<!--more -->

### Os alicerces para o desenvolvimento

Quando desenvolvemos um sistema embarcado a tarefa resumida consiste em:

1. Escrever o código do programa
2. Cross-compilar para o processador em uso
3. Gravar o microcontrolador
4. Testar/Debugar
5. Repetir exaustivamente até o resultado desejado, com ênfase na etapa 4 

Para executar essa lista de tarefas um conjunto de ferramentas é necessário.

A primeira tarefa exige um bom editor de texto, voltado para a programação. Eu já usei bastante coisa mas hoje 
trabalho basicamente no vim. Muitos editores de texto bons são produzidos para programadores então não vou nem 
tentar construir uma lista deles. Há muitas pela internet afora. 

Para traduzir o código, usualmente escrito em C ou seus dialetos, é necessário o uso de um compilador que levar
o seu texto a um idioma conhecido pelo processador, as suas instruções. No caso do Arduino que usa um microcontrolador 
com núcleo AVR o compilador utilizado é o avr-gcc.

Após compilado o código a imagem gerada deve ser gravada no microcontrolador. O microcontrolador possui uma memória
que não é apagada a cada vez que ele é reiniciado. Nos dias de hoje o tipo de memória é usualmente flash. Os 
microcontroladores proveem interfaces para realizar essa gravação, nos AVR um caminho possível é a interface
de [ICP( In-System Programming)](http://www.atmel.com/images/doc0943.pdf) através de um protocolo via SPI.  

Alguns microcontroladores oferecem também uma interface que permite executar o código linha a linha a fim de 
identificar possíveis problemas. No caso do AVR nem todos os microcontroladores possuem a interface específica 
para debug e então se faz necessário o uso de interfaces seriais, LED's e outros subterfúgios para localizar e 
corrigir os erros.

Para resolver estes problemas básicos o Arduino provê uma IDE(ambiente integrado de desenvolvimento), a interface 
de gravação é feita através de um bootloader serial(UART) e uma interface USB-Serial é disponibilizada nas placas. 
Para executar seções de debug no atmega328p é necessário um conjunto específico de ferramentas(AVR Dragon por exemplo) que 
utilizarão a interface debugWire. A IDE do Arduino não está preparada para executar seções de debug. 

### Executando um projeto sem o uso do Arduino ( com Makefile ) 

Deixando de lado a conversa e partindo pra ação. Serão necessárias as seguintes ferramentas:

* Um editor de texto( kate, notepad++, SublimeText, vim)
* avr-gcc ( winavr no windows )
* avrdude

Supondo que tudo está instalado comecemos um repositório. Um repositório é um local onde o seu código ficará armazenado e 
será versionado(nada de versao\_de\_sexta ou essefunciona ) para isso o github é uma boa opção. Não entrarei em 
detalhes sobre como criar um repositório no github já que isso é bem documentado na [internet](http://lmgtfy.com/?q=tutorial+github). 

Utilizaremos como receita de build um Makefile. O Makefile é um conjunto de instruções para realizar as tarefas
necessárias para que o código em C se transforme em binário. Para a criação do makefile pode ser usada a ferramenta
 [mfile](http://www.sax.de/~joerg/mfile/). No meu caso estou usando um [at90usb162](http://www.atmel.com/images/doc7707.pdf)
 de uma placa chamada [Micropendous1](https://code.google.com/p/micropendous/wiki/Micropendous1_Base) que tenho por aqui. 

```make 
MCU = at90usb162
FORMAT = ihex
TARGET = main
SRC = src/$(TARGET).c
ASRC = 
OPT = s
```   

Sem entrar muito em detalhes o primeiro parâmetro seleciona o microcontrolador em uso, SRC diz onde estão os fontes 
e OPT o nível de otimização do compilador. São os parâmetros que mais provavelmente será preciso mudar.

Gerado o Makefile é preciso criar o arquivo main.c, no diretório src(que deverá ser criado).

```C 
int main(void){
	}

```

O código acima é o básico pra compilar, entretanto ele não faz absolutamente nada além de prover a função main,
necessária no C. Mesmo o Arduino usa
[uma](https://github.com/arduino/Arduino/blob/master/hardware/arduino/avr/cores/arduino/main.cpp)
e reflete inclusive a estrutura que vou explorar na sequência.

Ao executar um projeto é importante selecionar uma estrutura. Nesse caso será implementada a estrutura mais básica 
em um projeto de sistema embarcado: o Powerloop.

### The mighty PowerLoop!

Em um sistema embarcado, uma vez iniciada a execução a tendência é que o mesmo fique funcionando o tempo todo 
aguardando a ocorrência de um evento para responder ao mesmo. Assim é quase certeza que você encontrará um loop 
infinito em algum lugar. 

Assim vou modificar o código pra fazer o simples pisca led. 


#### Configurando o hardware( Arduino setup() ) 

Uma coisa que eu gosto no Arduino é a ideia de separar as duas funções, setup() e loop() reforçando bem a ideia
de que há a necessidade de executar de modo infinito e que é necessário antes de tudo configurar o setup. Podemos 
encarar essa etapa inicial como sendo o equivalente a inicialização da bios de um computador desktop. 

Para que o LED pisque é preciso que um pino seja utilizado como saída. Pra isso vamos acrescentar uma seção de 
configuração do hardware w registradores no projeto, mas primeiro selecionaremos um pino.

O microcontrolador que estou utilizando é um [at90usb162](http://www.atmel.com/pt/br/Images/doc7707.pdf) é um 
microcontrolador com USB integrada(com bootloader que é uma mão na roda), alguns timers,um comparador e mais 
algumas interfaces. O [datasheet](http://www.atmel.com/pt/br/Images/doc7707.pdf) lista com mais detalhes.

Para configurar um pino como saída vamos escolher PC5.

A forma como os periféricos estão organizados faz que cada um deles possua um endereço reservado, um registrador. 
Estou utilizando o avr-gcc em conjunto com a [avr-libc](http://www.nongnu.org/avr-libc/) (o mesmo utilizado 
nas bibliotecas do Arduino) e assim vamos precisar de incluir no código a lib avr/io.h. Não entrarei nos detalhes
dela agora mas navegar no código pode ser uma boa pra entender o que eu disse sobre registradores e endereços. 

 
```C  
#include <avr/io.h>

#define PIN_USED 5

int main(void){
	// Configuração do PC5 como saída
	DDRC |= (1<<PIN_USED);
	//loop infinito	
	while(1){
		//Escrevendo 1 no pino 5 
		PORTC |= (1<<PIN_USED);
	}
}
```
O código apresenta os registradores que controlam a porta digital. Quando a função pinMode é chamada ao escrever
código para o Arduino no fim das contas o que será feito é setar esse registrador DDRx que é quem define qual a
direção que o pino será utilizado. O mesmo podemos dizer da função digitalWrite, ela irá realizar a escrita nesse 
registrador PORTx. É possível checar o código dessas funções no [repositório do Arduino no
github](https://github.com/arduino/Arduino/blob/master/hardware/arduino/avr/cores/arduino/wiring_digital.c).

Na linha 11 do código que liga o led temos a escrita exclusiva no bit PIN_USED do registrador PORTC. Do modo 
o código foi escrito a operação será realizada aplicando a máscara de bits gerada pelo deslocamento do valor 1
de PIN_USED posições e efetuando a operação ou bit a bit. O resultado é que somente o bit desejado é alterado.

Para que o led pisque é preciso que ele seja apagado. Entretanto antes de apagar o led é necessário
aguardar algum tempo pois o microcontrolador executa as instruções em uma taxa que tornaria
imperceptível o acender do led. 

```C

#include <avr/io.h>
#include <util/delay.h>

#define PIN_USED 5

int main(void){
	// Configuração do PC5 como saída
	DDRC |= (1<<PIN_USED);

	double ms_delay = 100;

    //loop infinito	
	while(1){
		//Escrevendo 1 no pino escolhido 
		PORTC |= (1<<PIN_USED);

		_delay_ms(ms_delay);
		
		PORTC &= ~(1<<PIN_USED);
	}
}

```

Na linha 19 temos a operação inversa zerando o bit da porta e apagando o led. Há ainda o uso da
função _delay_ms disponibilizada pela avr-libc. 

Propositalmente deixei não definida a macro F_CPU. A não definição e uso da função _delay_ms levou
ao seguinte warning como resultado da compilação. 

```no-highlight
# warning "F_CPU not defined for <util/delay.h>"
```

Houve sucesso e o código foi gerado mas não funcionaria corretamente dada a ausência do define. Isso
serve pra ilustrar que nem sempre gerar o código sem erros é sinal de que não há problemas e
certamente a ausência de um símbolo desses leva a problemas difíceis de localizar, vale como dica de
que a presença de warnings deve ser tradada com cuidado. 

A solução do problema se dá com a modificação do Makefile e acrescentando F_CPU no valor da
frequência e modificando a linha contendo CDEFS, que é passado ao compilador. 

``` Makefile 
CDEFS = -DF_CPU=$(F_CPU)
``` 

Assim o código é compilado e o hex é gerado sem warnings e executará conforme planejado. O resultado
da compilação após o comando make é:


``` no-highlight
avr-gcc -c -mmcu=at90usb162  -I. -gstabs -DF_CPU=8000000UL  -Os -Wall -Wstrict-prototypes -std=gnu99
src/main.c -o src/main.o 
avr-gcc -mmcu=at90usb162  -I. -gstabs -DF_CPU=8000000UL  -Os -Wall -Wstrict-prototypes -std=gnu99
src/main.o   --output main.elf     -lm
avr-objcopy -O ihex -R .eeprom main.elf main.hex
avr-objcopy -j .eeprom --set-section-flags=.eeprom="alloc,load" \
--change-section-lma .eeprom=0 -O ihex main.elf main.eep
avr-objcopy: --change-section-lma .eeprom=0x0000000000000000 never used

Size after:
AVR Memory Usage
----------------
Device: at90usb162

Program:     170 bytes (1.0% Full)
(.text + .data + .bootloader)

Data:          0 bytes (0.0% Full)
(.data + .bss + .noinit)
``` 

O objetivo desse texto é dar uma visão de como é o desenvolvimento utilizando o AVR sem utilizar as
bibliotecas do Arduino. Naturalmente é possível, inclusive aconselhável a escrita de funções para
reaproveitar código e mesmo organizar melhor o projeto e acelerar o desenvolvimento. 

Outro ponto que é importante destacar é que não há qualquer intensão em defender o não uso do
Arduino. A ideia é simplesmente ilustrar outras opções. 

O código produzido está neste [repositório](https://github.com/euripedesrocha/noarduino).

<div class="fb-share-button"
data-href="https://parumtronics.github.io/blog/equandonaohaviaoarduino.html "
data-layout="button_count"></div>

