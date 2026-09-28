---
tipo: sistema
data: 2026-09-28
area: gameplay
arquivos: [src/player/flight.js, src/world/collision.js, src/fx/breach.js, src/fx/debrisSim.js]
tags: [sistema, destruicao, gameplay]
---

# Destruição de prédios

![[destruicao-furos.jpg]]

## O que é

Voando rápido contra um prédio, o herói **fura a fachada, atravessa e sai do outro lado**,
e deixa um rombo na entrada, outro na saída, entulho de concreto e vidro e poeira. Devagar,
o prédio continua sólido: ele para ou desliza na parede, como antes.

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

- **Furo**: decalque com o rombo desenhado (a parede é uma casca oca, então o buraco não
  é recortado), alinhado ao eixo da face. O tamanho cresce com a velocidade (~6–8 m a
  150 m/s). Pool de 160.
- **Entulho** (`fx/debrisSim.js`, lógica pura testada): pool de 1.200 pedaços em arrays
  planos, com gravidade, quique com atrito, giro e sono ao parar. Na saída, o material
  empurrado sai a 55–110% da velocidade do herói e voa junto com ele; na entrada, a
  fachada estoura para fora. 80% concreto, 20% vidro.
- **Poeira**: `Points` com tamanho e opacidade por nuvem, num shader próprio.
- **Poeira na tela**: enquanto a câmera está dentro de um prédio (`heightAt` da câmera
  acima dela), o HUD cobre a tela de poeira. As paredes são de face única e, lá de dentro,
  a cidade apareceria "de raio-x".
- Som de parede cedendo (`audio.breach`: estalo seco na entrada, estrondo na saída) e
  tremor de câmera.

## Parâmetros que importam

| Parâmetro | Valor | Por quê |
|---|---|---|
| `SMASH.speed` | 30 m/s para dentro da parede | O cruzeiro (48 m/s) de frente fura; raspar não |
| `SMASH.entry` | 6 m/s | A fachada resiste |
| `SMASH.perMeter` | 0,012 por m (× v) | Um prédio de 30 m tira ~30% |
| `SMASH.min` | 10 m/s | Nunca fica preso dentro |
| Entulho | 1.200 pedaços, sono < 0,8 m/s | Cabe no orçamento de 60 fps |

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

## PRs

- [[2026-09-28-pr-021-atravessar-predios]]
