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

### Desabamento

**Dano** (`world/damage.js`, lógica pura testada). Cada entrada de travessia tira do andar
`rombo (6 m) / largura atravessada × (1 + v/200)`, somado por faixa de 8 m de altura.
Quando uma faixa chega a 50%, tudo acima da base dela desaba. Na prática:

- supersônico (420 m/s) num prédio de até ~37 m de largura cai de uma vez;
- em cruzeiro, são precisas duas passadas na mesma altura;
- passadas em alturas diferentes não somam, porque é o mesmo andar que precisa ceder.

O Planeta Diário fura, mas não cai, porque o globo e o letreiro não acompanhariam.

**Corte da malha** (`world/buildingGeo.js`, `cutTier`, testado contra a geometria real).
Cada face de parede é um quadrilátero do chão ao topo do nível, com o `v` da textura
proporcional à altura. Cortar é baixar os dois vértices de cima e o `v` deles juntos, então
as janelas não esticam. O telhado desce para o corte, a cornija some, e um nível inteiro
acima do corte vira um ponto (triângulo de área zero). A cidade guarda, por prédio, o
primeiro vértice de cada nível em cada malha mesclada (`city.refs`). Só essas faixas sobem
para a GPU (`addUpdateRange`).

**Queda** (`world/collapse.js`):

- `city.makePart` remonta a parte de cima idêntica (mesmo deslocamento de janelas, mesmo
  tom, montada com y absoluto para o `v` bater) e ela cai com 7 m/s² efetivos, tombando
  até ~8°;
- a linha de esmagamento (base do bloco) encurta o prédio no lugar a cada quadro e baixa o
  teto das caixas de colisão;
- os objetos de telhado do prédio caem junto: mesma transformação do bloco;
- dos lados, na linha de esmagamento, sai poeira grande e entulho;
- o bloco afunda sob a laje da rua, que é opaca e o esconde. No fim sobram o toco de 3 m e
  escombros (70 pedaços grandes), e furos e marcas de queimado da parte que caiu somem.

## Parâmetros que importam

| Parâmetro | Valor | Por quê |
|---|---|---|
| `SMASH.speed` | 30 m/s para dentro da parede | O cruzeiro (48 m/s) de frente fura; raspar não |
| `SMASH.entry` | 6 m/s | A fachada resiste |
| `SMASH.perMeter` | 0,012 por m (× v) | Um prédio de 30 m tira ~30% |
| `SMASH.min` | 10 m/s | Nunca fica preso dentro |
| Entulho | 1.200 pedaços, sono < 0,8 m/s | Cabe no orçamento de 60 fps |
| Queda | 7 m/s² efetivos, tombando até ~8° | Mais lenta que queda livre: os andares freiam |
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
