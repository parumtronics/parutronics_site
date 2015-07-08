---
title: "Projetando uma placa microcontrolada do zero: Parte 1"
layout: page
date: 2014-05-15
---

Indo direto ao ponto: Essa é uma série de posts sobre como levar uma ideia de
projeto, que eventualmente já foi prototipada usando um Arduino e shields, a
uma placa dedicada ao projeto. Na primeira parte trataremos do básico, estrutura
e seleção de componentes.

### Planejando a estrtutra básica do projeto

Então você teve uma ideia e está empolgado em trazê-la a vida, bem vindo ao
clube. Não existe uma receita definitiva para atingir o estágio de protótipo.
Você pode até considerar aqueles vários shields em cima de um arduino como um
protótipo, mas aquele cenário está mais para uma prova de conceito. Um protótipo
é um primeiro passo para a produção de uma placa que eventualmente se tornará um
produto, ou não.

O primeiro passo é estabelecer o que é necessário na placa. Usualmente nós vamos
partir de um conjunto de requisitos da sua placa uma descrição como:

"Essa placa é uma placa de desenvolvimento compatível com o novo Arduino zero,
porém possuindo o mesmo formato do arduino nano"

Sim é isso que faremos ;)

Outras placas irão inserir outras descrições. Eu não gosto de inserir coisas do
tipo

"Essa placa(produto) utiliza um lm35"

Ao invés disso:
"Essa placa(produto) mede a temperatura... "

#### Diagrama de blocos

A partir de uma descrição do que se deseja para o projeto é interessante a
construção de um diegrama de blocos. O diagrama de blocos dará uma visão de como
o hardware estará distribuído na placa. Eu utilizo o [Kicad](http://www.kicad-pcb.org)
no desenvolvimento do hardware dos meus projetos e nessa primeira etapa eu
utilizo a ideia de folhas hierárquicas e de anotações no circuito para definir
cada uma das partes. O resultado é depois aproveitado para o desenvolvimento de
esquemáticos, em uma placa mais complexas vamos explorar um pouco mais disso,
deixe uma sugestão para o próximo projeto nos comentários.

### Seleção do microcontrolador

Um ponto muito importante é selecionar corretamente o microcontrolador com as
características necessárias ao seu projeto. E não, o fato de o atmega328p estar
no arduino não o torna o escolhido para todos os problemas. Nossos amigos do
Embarcados traçaram uma discussão sobre o assunto então não me estenderei no
mesmo. Mas os pontos mais relevantes na seleção são:

- **Adequação ao projeto** - Os periféricos atendem as conexões necessárias? Há
  poder de processamento suficiente pra executar os algoritmos que
  possivelmente serão executados? A potência consumida é adequada ao
  projeto?

- **Ferramentas** - Há ferramentas gratuitas e de qualidade? Há um debugger de
  baixo custo no mercado?

- **Experiência anterior/código legado** - Há tempo para desenvolver as
  habilidades necessárias para a nova escolha? Há uma quantidade de código
  escrita para um outro microcontrolador?

Seções típicas de uma placa em sua rev0
Particionamento de hw/sw
coleta de email para acompanhamento da parte 2: o circuito de alimentação


