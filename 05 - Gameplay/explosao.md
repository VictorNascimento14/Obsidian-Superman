---
tipo: sistema
data: 2026-09-28
area: gameplay
arquivos: [src/fx/fireSim.js, src/fx/explosion.js, src/audio/audio.js, src/main.js]
tags: [sistema, destruicao, efeitos]
---

# Explosão

![[explosao.jpg]]

## O que faz

Furar a fachada de um prédio explode, na entrada e (maior) na saída. Vêm, nesta ordem:

- um **clarão** que ilumina a rua;
- a **onda de choque**, uma casca de borda acesa que cresce em meio segundo;
- a **bola de fogo**, que sai da fachada e sobe: branco-amarela, depois laranja e vermelho
  escuro;
- as **brasas** voando em arco;
- a **fumaça escura**, que nasce no lugar do fogo e sobe em rolos por 5 a 9 s;
- um **estrondo** grave.

Tudo cresce com a força do golpe: a velocidade, multiplicada pela
[[carga-solar]]. A 150 m/s é uma bola de fogo com uns 12 m de raio; supersônico (ou carregado),
com mais de 25 m. Escolha do jogador na fase 3: clarão, fogo e fumaça.

## Como funciona

**Partículas** (`fx/fireSim.js`, lógica pura testada):

- `createParticles(n, { gravity, buoyancy, drag })`: pool em anel em arrays planos, com
  posição, velocidade, idade, vida e tamanho inicial e final;
- a fração da vida é negativa enquanto a partícula espera para aparecer: é assim que a fumaça
  nasce um pouco depois do fogo;
- o tamanho cresce com a raiz da fração: rápido no começo, como uma explosão;
- `blastSize(power)`: quanto fogo, fumaça e brasa, o raio da bola de fogo e o clarão, para
  uma força de 0 a 1.

**Desenho** (`fx/explosion.js`):

- três `Points` (fogo aditivo em HDR, fumaça com transparência, brasas aditivas) que leem a
  posição direto do array da simulação, sem cópia por quadro;
- fogo e fumaça usam a mesma textura de nuvem (borrões radiais num canvas, com semente fixa),
  girada por partícula para os puffs não saírem iguais;
- a fumaça tem um tom por puff (a semente) e escurece à noite (`light`); o fogo não;
- o clarão é **uma** `PointLight`, reaproveitada. Com o número de luzes fixo, os shaders da
  cidade não recompilam a cada explosão;
- a onda de choque é uma esfera com só a borda acesa (fresnel na 6ª potência).

**No jogo** (`main.js`): em cada ruptura, `blast(ponto + normal × 3 m, normal, força)`, com
força = `velocidade × (1 + 2 × carga) / 420`, a entrada a 80%. O fogo é soprado para fora da
fachada.

## Parâmetros que importam

| Parâmetro | Valor | Efeito |
|---|---|---|
| Pools | fogo 320 · fumaça 260 · brasas 400 | Várias explosões ao mesmo tempo |
| Fogo por explosão | 26 → 72 | Pela força |
| Raio da bola de fogo | 7 → 26 m | Pela força |
| Vida | fogo 0,8–1,7 s · fumaça 5–9 s · brasa 1–2,6 s | A fumaça fica depois |
| Clarão | 6.000 cd no pico, cai a 9/s | Ilumina a rua de noite |
| Centro | 3 m para fora da fachada | O prédio não esconde metade do fogo |

## Armadilhas

- **Fogo nascendo na parede some pela metade.** Com o centro a 1,5 m da fachada e as
  partículas espalhadas em todas as direções, metade ia para dentro do prédio e o fogo virava
  uma meia-bola cortada. O centro foi para 3 m, e o sopro para fora é mais forte.
- **Poucos puffs grandes viram uma bola chapada.** A primeira fumaça eram poucas partículas
  enormes sobrepostas: um disco cinza uniforme. Agora são mais, menores, espalhadas pelo volume
  do fogo e cada uma num tom.
- **A casca da onda de choque inteira parecia uma bolha de vidro** à noite. Fica só a borda.
- **Câmera de screenshot dentro do prédio vizinho**: a 75 m da fachada já é o quarteirão da
  frente. O script põe a câmera no eixo da rua. Para isso existe o gancho de depuração
  `__game.camAt(pos, alvo)`, uma câmera parada.

## PRs

- [[2026-09-28-pr-029-explosao]]
