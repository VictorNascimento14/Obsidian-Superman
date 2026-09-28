---
tipo: sistema
data: 2026-09-28
area: gameplay
arquivos: [src/player/flight.js, src/world/collision.js, src/fx/breach.js, src/fx/debrisSim.js, src/world/damage.js, src/world/collapse.js, src/world/buildingGeo.js]
tags: [sistema, destruicao, gameplay]
---

# Destruição de prédios

![[destruicao-furos.jpg]]

## O que é

Voando rápido contra um prédio, o herói **fura a fachada, atravessa e sai do outro lado**,
e deixa um rombo na entrada, outro na saída, entulho de concreto e vidro e poeira. Devagar,
o prédio continua sólido: ele para ou desliza na parede, como antes.

Dano grande num andar **derruba o prédio**: a parte de cima despenca sobre a de baixo e a
esmaga até o chão, numa nuvem de poeira, como numa demolição. Sobra um toco com escombros.

![[desabamento.jpg]]

![[queda-realista.jpg]]

## Como funciona

**Decisão no contato** (`flight.js`, testado). O `onContact` da [[colisao]] recebe a
normal e o índice da caixa, e decide **antes** do empurrão:

- a caixa é de prédio (`breakable`), o herói está no ar e a componente da velocidade
  **para dentro** da parede é ≥ `SMASH.speed` (30 m/s): o contato devolve `true` e a caixa
  não empurra. Raspar de lado (componente pequena) continua deslizando;
- a caixa entra em `smashing`, uma lista fixa de 4 vagas que a colisão ignora até o herói
  sair. Quebrar a fachada custa `SMASH.entry` (6 m/s);
- a cada subpasso lá dentro, cada metro custa `perMeter × v` (decaimento exponencial com
  a distância: um prédio de 30 m tira ~30% da velocidade em qualquer marcha). A velocidade
  nunca cai abaixo de `SMASH.min` (10 m/s): quem entrou sai do outro lado;
- ao sair, o ponto da caixa mais perto do centro está na face de saída: é onde fica o
  furo de saída.

Cada entrada e saída vira um evento `breach` com ponto, normal, direção e velocidade.

**Efeitos** (`fx/breach.js`):

- **Furo**: decalque com o rombo desenhado, alinhado ao eixo da face. O tamanho cresce com a
  velocidade (~6–8 m a 150 m/s). Pool de 160. Nos prédios com interior montado o furo é
  **aberto de verdade**: a parede e o miolo do decalque são recortados no shader, e o andar
  aparece lá dentro — ver [[interior-dos-predios]].
- **Entulho** (`fx/debrisSim.js`, lógica pura testada): pool de 1.200 pedaços em arrays
  planos, com gravidade, quique com atrito, giro e sono ao parar. Na saída, o material
  empurrado sai a 55–110% da velocidade do herói e voa junto com ele; na entrada, a
  fachada estoura para fora. 80% concreto, 20% vidro.
- **Poeira**: `Points` com tamanho e opacidade por nuvem, num shader próprio.
- **Poeira na tela**: enquanto a câmera está dentro de um prédio (`heightAt` da câmera
  acima dela), o HUD cobre a tela de poeira. Com o interior montado, a poeira fica em 12%,
  porque lá dentro há o que ver; sem ele, as paredes de face única mostrariam a cidade
  "de raio-x".
- **Interior**: lajes, pilares, núcleo, salas, mesas e luminárias em volta de onde o herói
  entrou. O que ele toca quebra e vira entulho na cor do material — ver
  [[interior-dos-predios]].
- Som de parede cedendo (`audio.breach`: estalo seco na entrada, estrondo na saída) e
  tremor de câmera.
- **Explosão**: clarão, onda de choque, bola de fogo, brasas e fumaça saindo da fachada, maiores
  com a força do golpe — ver [[explosao]].

### Desabamento

**Dano** (`world/damage.js`, lógica pura testada). Cada entrada de travessia tira do andar
`rombo (6 m) / largura atravessada × (1 + v/200)`, somado por faixa de 8 m de altura.
Quando uma faixa chega a 50%, tudo acima da base dela desaba. Além disso, **golpe com força
de supersônico derruba qualquer prédio de uma vez** (`DAMAGE.topple` = 340). A força é a
velocidade vezes a [[carga-solar]]: 150 m/s carregado também derruba. Foi escolha do jogador
na fase 3. Na prática:

- supersônico (ou carregado): a parte de cima cai, em qualquer largura;
- em cruzeiro, são precisas duas passadas na mesma altura num prédio estreito;
- passadas em alturas diferentes não somam, porque é o mesmo andar que precisa ceder.

**Ceder em volta do furo.** Furo que não derruba (entrada ou saída) faz os andares em volta
dele cederem, 0,3 a 0,65 s depois (evento `cede`):

- tudo do interior numa esfera de 4 a 7 m cai, laje inclusive — por isso a laje vem em placas
  de 6 m ([[interior-dos-predios]]);
- a fachada abre num rombo de vários andares (uma abertura maior);
- blocos grandes de concreto despencam para a rua, com poeira.

O Planeta Diário fura, mas não cai, porque o globo e o letreiro não acompanhariam.

**Corte da malha** (`world/buildingGeo.js`, `cutTier`, testado contra a geometria real).
Cada face de parede é um quadrilátero do chão ao topo do nível, com o `v` da textura
proporcional à altura. Cortar é baixar os dois vértices de cima e o `v` deles juntos, então
as janelas não esticam. O telhado desce para o corte, a cornija some, e um nível inteiro
acima do corte vira um ponto (triângulo de área zero). A cidade guarda, por prédio, o
primeiro vértice de cada nível em cada malha mesclada (`city.refs`). Só essas faixas sobem
para a GPU (`addUpdateRange`).

**Queda** (`world/fall.js`, lógica pura testada; `world/collapse.js`, o desenho):

- a parte de cima vira uma pilha de até 5 segmentos de ~30 m. Cada um é uma cópia idêntica do
  trecho do prédio (`city.makePart(bi, y0, y1)`: mesmas janelas, mesmo tom), com uma tampa de
  concreto quebrado onde o corte cai no meio de um nível. Sem ela, um segmento solto seria uma
  casca oca;
- primeiro o bloco inteiro desce a 6 m/s², esmagando os andares de baixo. Ao mesmo tempo ele
  tomba **para o lado do golpe**, girando sobre a borda da base daquele lado, cada vez mais
  rápido, como uma árvore;
- em 1,4 s os segmentos se soltam, cada um com a velocidade que tinha, um afastamento e um giro
  próprios, e caem como corpos rígidos (7 m/s²);
- cada segmento que chega ao chão (ou ao telhado de um vizinho) afunda um quarto da altura
  nele e se desfaz. Viram pedaços grandes e uma **nuvem de poeira em anel** que corre pelas
  ruas e fica no ar por 14 a 20 s;
- o toco é esmagado pelo segmento de baixo até 3 m, com poeira e entulho saindo pelos lados na
  linha de esmagamento. As caixas de colisão acompanham;
- os objetos de telhado caem com o segmento de cima;
- no fim sobram o toco de 3 m e escombros, e furos e marcas de queimado da parte que caiu
  somem.

## Parâmetros que importam

| Parâmetro | Valor | Por quê |
|---|---|---|
| `SMASH.speed` | 30 m/s para dentro da parede | O cruzeiro (48 m/s) de frente fura; raspar não |
| `SMASH.entry` | 6 m/s | A fachada resiste |
| `SMASH.perMeter` | 0,012 por m (× v) | Um prédio de 30 m tira ~30% |
| `SMASH.min` | 10 m/s | Nunca fica preso dentro |
| Entulho | 1.200 pedaços, sono < 0,8 m/s | Cabe no orçamento de 60 fps |
| Queda | inteira 1,4 s (6 m/s², tombo até 0,9 rad), depois segmentos a 7 m/s² | Tomba para o lado do golpe e se parte no ar |
| Derrubar de uma vez | força ≥ 340 | Supersônico ou carregado derruba qualquer prédio |
| Ceder | esfera de 4–7 m, 0,3–0,65 s depois | Andares em volta do furo caem em pedaços |
| Dano para desabar | 50% de uma faixa de 8 m | Supersônico derruba prédio estreito de uma vez |
| Toco | 3 m | O térreo fica, com escombros |
| Carga solar | `might = 1 + 2 × carga` | Fachada e metros freiam `might` vezes menos; o dano usa `force = v × might` |

Com a [[carga-solar]] cheia, a 150 m/s um prédio de 24–34 m cai de uma vez (sem carga, o
mesmo golpe tira ~0,3 da faixa e só fura). A velocidade para furar continua 30 m/s.

## Armadilhas

- **Empurrar antes de decidir cria laço**: o `pushOut` deixava a esfera tangente à face
  antes de avisar o contato; no subpasso seguinte ela parecia "já ter saído", e o herói
  entrava e saía da mesma parede a cada subpasso. O contato decide antes do empurrão.
- **`centro − normal·raio` não está na fachada** quando a esfera já penetrou: o furo ficava
  dentro da parede. Use o ponto da caixa mais perto do centro.
- **Dentro do prédio, `heightAt` é o telhado**: o pouso teleportava o herói 30 m acima. Não
  há pouso enquanto houver caixa em `smashing`.
- **Entulho: parede × chão.** Chão novo acima de onde o pedaço *estava* é parede de prédio,
  e o pedaço rebate. Comparar com o y *novo* confundia uma queda rápida com uma parede.
- **Caixa de altura zero é uma placa invisível no ar.** O nível que sumiu no desabamento
  vai para debaixo da terra (`minY = maxY = −10⁴`), não para `maxY = minY`.
- **Marcas coladas em prédio que caiu ficam no ar**: furos e queimados na pegada, acima do
  corte, são escondidos (`hideInstancesIn`).
- **Montar a parte que cai com y relativo desloca as janelas**: monte com y absoluto e
  translade depois.

## PRs

- [[2026-09-28-pr-021-atravessar-predios]]
- [[2026-09-28-pr-022-desabamento]]
- [[2026-09-28-pr-026-carga-solar]] — força e freio com a carga solar
- [[2026-09-28-pr-028-interior-dos-predios]] — interior, furo aberto de verdade, peças que quebram
- [[2026-09-28-pr-029-explosao]] — explosão em cada ruptura
- [[2026-09-28-pr-030-queda-realista]] — rápido derruba, devagar cede; tombo, segmentos, nuvem de poeira
- [[2026-09-28-pr-031-visao-corta]] — a visão de calor fatia prédios (corte limpo) e estoura furos
