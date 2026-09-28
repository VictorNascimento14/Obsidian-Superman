---
tipo: sistema
data: 2026-09-28
area: mundo
arquivos: [src/world/interior.js, src/world/interiors.js, src/world/openings.js, src/world/buildingGeo.js, src/world/city.js, src/fx/breach.js, src/main.js]
tags: [sistema, mundo, destruicao, render]
---

# Interior dos prédios

![[interior-dos-predios.jpg]]

## O que faz

Os prédios eram cascas ocas, com as janelas pintadas na textura: atravessando um, a câmera via
o vazio (e a cidade "de raio-x"). Agora, quando o herói fura um prédio, ele ganha interior em
volta de onde o herói entrou:

- lajes, com piso de carpete e forro claro;
- pilares numa grade alinhada às janelas;
- o núcleo de elevadores em concreto;
- divisórias e salas, ou planta livre com fileiras de mesas e armários;
- luminárias no forro, acesas;
- o lado de dentro da fachada, com as janelas e o céu lá fora.

O furo vira abertura de verdade: por ele se vê o andar lá dentro. Passando, o herói arrebenta
divisórias, mesas, armários, luminárias e pilares no caminho, e eles viram entulho na cor
do material.

## Como funciona

**Gerador** (`world/interior.js`, lógica pura testada):

- `interiorBand(b, yLo, yHi)` devolve as peças (caixas com tipo e id) dos andares com base na
  faixa pedida;
- cada andar sai de uma semente própria (prédio + andar). O mesmo pedaço tem o mesmo id em
  qualquer faixa, e o que quebrou continua quebrado quando a faixa muda;
- as lajes são do prédio, não do andar: uma em cada divisa de andar e uma embaixo do telhado
  de cada nível. Num recuo, fica a do nível de baixo, que é maior. Elas vêm em placas de 6 m,
  para os andares em volta de um furo cederem em pedaços (`collapseRegion`,
  [[2026-09-28-pr-030-queda-realista]]);
- com núcleo (planta de 16 m ou mais), 45% dos andares são de salas: um corredor em volta do
  núcleo, divisórias até a fachada e uma mesa por sala. O resto é planta livre, com mesas em
  fileiras e armários encostados no núcleo;
- paredes vêm em segmentos de 3 m, para quebrar um pedaço de cada vez;
- `piecesInSphere` diz o que uma esfera toca.

**Desenho** (`world/interiors.js`):

- um `InstancedMesh` por tipo de peça, com capacidade fixa;
- interior só nos 3 últimos prédios furados, e só ±3 andares em volta de onde o herói entrou.
  Se ele mergulha lá dentro, a faixa acompanha;
- a lista de prédios só muda num furo, e aí as instâncias são reescritas; o que quebra
  some sozinho, e só aquela instância sobe para a GPU (`addUpdateRange`);
- lá dentro o céu chega só pelas janelas: a luz do hemisfério (o azul de cima, que não tem
  sombra) cai a um quarto, e o reflexo do ambiente a 0,3;
- a fachada por dentro desenha as janelas no shader, em vãos de 4 m, com o céu atrás. O céu
  fica abaixo de 1 no HDR para não virar bloom, e escurece à noite.

**Abertura de verdade** (`world/openings.js`):

- até 24 furos ao mesmo tempo, em uniforms compartilhados pelos materiais das paredes, do
  lado de dentro da fachada e do decalque do furo;
- dentro de um furo o fragmento é descartado, com a borda recortada por senos sobre o ângulo
  em volta do centro;
- nas paredes mescladas, só os prédios com interior montado testam os furos: o atributo por
  vértice `open` fica em 1 nas faixas deles (`city.setOpen`). O resto da cidade não paga nada;
- o decalque do furo (`fx/breach.js`) também é recortado. O miolo escuro dele some junto com a
  parede, e fica a borda de concreto quebrado. O raio da abertura é 30% do tamanho do
  decalque, o mesmo do miolo desenhado na textura;
- prédio que sai da lista (o quarto furado, ou o que desabou) fecha os furos: o decalque
  volta a mostrar o miolo escuro.

**No jogo** (`main.js`):

- furo de entrada: monta o interior e abre o furo (a saída também abre);
- enquanto `flight.inside`, a esfera de 2,2 m em volta do caminho do herói quebra o que toca,
  com entulho e poeira;
- prédio que desaba perde o interior;
- com interior montado, a poeira na tela fica em 12% (antes cobria tudo).

## Parâmetros que importam

| Parâmetro | Valor | Efeito |
|---|---|---|
| Prédios com interior | 3 | Os últimos furados |
| Faixa | ±3 andares | Em volta de onde o herói entrou |
| Grade de pilares | 8 m | Dois vãos de janela |
| Núcleo | 26% da planta (mínimo 6 m) | Só em planta de 16 m ou mais |
| Raio de quebra | 2,2 m | Pega mesa e luminária em qualquer altura do andar |
| Furos abertos | 24 | Uniforms do shader |

Medido: o maior prédio (82 × 82 m) dá 4.479 peças em ±3 andares, geradas em 6,8 ms; um médio,
729. Atravessando a 45 m/s: 16,7 ms por quadro (preso no vsync), um único pico de 33 ms no
impacto; 120 draw calls, 466 mil triângulos.

## Armadilhas

- **A parede de face única não fazia sombra no próprio prédio.** Por padrão o three.js desenha
  no mapa de sombra só a face de trás de um material de face única. Para a luz, quem aparecia
  era a parede do fundo, e o miolo do prédio ficava ao sol: o interior saía claro e lavado. As
  paredes passaram a entrar no mapa de sombra dos dois lados (`shadowSide`). Na cidade vista de
  cima, antes e depois são iguais (o `normalBias` de 0,6 segura o serrilhado).
- **A luz do hemisfério não tem sombra.** Dentro de um andar, o azul do céu chegava pelo teto
  e tingia o carpete. O shader do interior divide essa luz por quatro.
- **Janela de dentro acima de 1 no HDR** virava um véu de bloom na tela inteira.
- **Nenhum prédio da cidade tem menos de 16 m de lado**: o teste do prédio sem núcleo usa um
  prédio sintético.

## PRs

- [[2026-09-28-pr-028-interior-dos-predios]]
