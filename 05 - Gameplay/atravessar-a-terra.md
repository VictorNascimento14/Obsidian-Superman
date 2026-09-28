---
tipo: sistema
data: 2026-09-28
area: gameplay
arquivos: [src/powers/pierce.js, src/powers/heatvision.js, src/space/space.js, src/main.js]
tags: [sistema, poderes, espaco, visao-de-calor]
---

# Atravessar a Terra

![[terra-atravessada.jpg]]

## O que faz

Com a [[carga-solar]] (a partir de 35%), a [[visao-de-calor]] que acerta a Terra atravessa o
planeta: entra de um lado, passa pelo miolo e sai do outro, deixando um buraco em brasa na
entrada e outro na saída. Os buracos crescem enquanto o raio fica neles (até 400 km de
raio) e continuam lá pelo resto da sessão. É o pedido: "se eu usar a visão de calor na Terra
estando perto do Sol, o poder é tão grande que atravessa a Terra e deixa o buraco".

Do Sol, a Terra tem 9 px: com o ◇ TERRA a até ~3° da mira, o raio vai sozinho ao centro dela
(mira assistida). De perto, vai onde o jogador mira.

## Como funciona

**Lógica** (`powers/pierce.js`, pura e testada):

- `assistAim(o, dir, out)`: no disco da Terra, mantém a mira; a até `PIERCE.assist` da borda,
  vai ao centro. O ângulo é medido por `atan2(|a × b|, a · b)`, que não perde precisão perto
  de zero;
- `chord(o, d, out)`: as distâncias de entrada e de saída do raio pela esfera. A distância de
  passagem é medida direto (o vetor da origem menos a projeção). Do Sol, `b² − c` teria 22
  casas, e a diferença (o raio da Terra ao quadrado, 13 casas) sumiria;
- `createPierce().update(dt, o, dir, charge, firing)`: o disparo do quadro, com a direção,
  a entrada, a saída, `blocked` e o buraco. Ele abre um buraco novo ou alarga o que está sob o
  raio. Um buraco novo só abre se a entrada estiver a mais de `max(2 × raio, 250 km)` dos
  outros; senão o existente cresce e desliza atrás do raio (`follow`). Assim o ajuste da
  câmera de ombro ao começar a mirar não vira uma fila de buracos.
- **Metrópolis é protegida**: a menos de 150 km dela, o raio para na superfície (aviso
  METRÓPOLIS, NÃO). A cidade de verdade é plana e não teria como mostrar o buraco.

**No jogo** (`main.js`):

- o raio sai da câmera, na direção da mira, como o da visão de calor de sempre;
- a visão de calor da cidade recebe a direção assistida (`update(dt, wants, over)`), para o
  feixe perto do herói apontar para a Terra;
- o disparo é decidido no próprio quadro, com a mesma regra da reserva. Com o `firing` do
  quadro anterior, um teleporte logo depois de soltar o botão abria um buraco onde a câmera
  caísse;
- buraco novo: aviso **A TERRA FOI ATRAVESSADA!** e um estrondo.

**Render** (`space/space.js`):

- o raio na cena do espaço: um cone do herói até a entrada, com a ponta no herói, e um
  cilindro da saída até dois raios da Terra além dela. O raio de cada um é proporcional à
  distância, e a espessura aparente fica ~constante (0,0012 rad) de perto até 1 UA;
- os buracos no shader do globo: até 8, cada um com entrada e saída
  (`holes[16]`, direção geográfica e `1 − cos(raio)`). O túnel é escuro no fundo, com a parede
  em brasa; a borda derretida em HDR (o bloom a faz brilhar, também de noite); e o chão
  queimado em volta, até 3 raios.

## Parâmetros que importam

| Parâmetro | Valor | Efeito |
|---|---|---|
| `PIERCE.min` | 35% de carga | Menos que isso, a visão de calor não atravessa |
| `PIERCE.assist` | 0,05 rad (~3°) | Tolerância da mira em volta do disco |
| `PIERCE.start` / `grow` / `max` | 25 km / 40 km/s / 400 km | Buraco ao abrir, crescimento, teto |
| `PIERCE.merge` / `follow` | 250 km / 4 por s | A mira que anda continua no mesmo buraco, que a segue |
| `PIERCE.count` | 8 | Buracos guardados (16 pontos no shader) |
| `PIERCE.city` | 150 km | Metrópolis protegida |

## Armadilhas

- **A mira é a câmera, não o rumo do herói.** A câmera de ombro olha uns 4° abaixo do rumo:
  mirando pelo rumo, do Sol, o raio passava fora da Terra. Os scripts de teste corrigem a mira
  pela direção da câmera, como o jogador faz com o marcador.
- **`firing` do quadro anterior** abria um buraco onde a câmera caísse depois de um teleporte.
- **Buraco por quadro**: o ajuste da câmera ao começar a mirar anda a entrada centenas de km
  em frações de segundo, e o critério "dentro do raio do buraco" criava 8 buracos em fila.
- **Na cidade, o raio carregado para no chão.** Sem limitar ao mundo fora da cidade, disparar
  a visão de calor carregada contra um prédio seguia a esfera por baixo, caía no raio protegido
  e mostrava "METRÓPOLIS, NÃO" a cada disparo. No mundo plano (`nearCity`), a Terra não é
  atravessada.
- **A mira gira ~1° ao começar a disparar.** A câmera vai para o ombro (e o pitch do voo vai só
  até 83°: de cima da cidade a mira nem chega nela). Mirando Metrópolis de 14.000 km, o ponto no
  chão andou de 6 km para 300 km da cidade em 1 s, saiu do raio protegido e abriu um buraco; de
  1.000 km, anda 30 km. No jogo a mira do HUD anda junto, e o jogador vê onde o raio cai. O e2e
  mira a cidade de 1.000 km, a 45°.

## PRs

- [[2026-09-28-pr-027-visao-atravessa-terra]]
