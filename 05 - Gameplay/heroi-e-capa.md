---
tipo: sistema
data: 2026-09-28
area: gameplay
arquivos: [src/player/hero.js, src/player/cloth.js]
tags: [sistema, heroi, capa, animacao]
---

# Herói e capa

![[heroi-poses.jpg]]

## O que faz

Modelo do herói montado com primitivas (ADR-001), animado por poses combinadas, e
uma capa simulada como pano.

## Como funciona

**Rig** (`hero.js`) — origem na pélvis; em pé, a sola fica `FLIGHT.footDepth` (1,04 m,
definido em `flight.js`: a física é lógica pura e não importa o modelo — ADR-001)
abaixo. Corpo olha para +z, cabeça para +y. Juntas são `Object3D`: pélvis → coluna →
peito → pescoço → cabeça; ombro → cotovelo → mão; quadril → joelho → tornozelo.
Escala 1,1 → ~1,9 m.

**Poses** — ângulos de Euler por junta; cada pose tem um peso que vai ao alvo com
suavização exponencial (`1 − e^(−6·dt)`), e a rotação final é a média ponderada.
Por cima: respiração, passada (quando `walk > 0`) e ondulação em voo.

| Pose | Quando |
|---|---|
| `idle` | em pé, mãos na cintura |
| `hover` | pairando: um joelho dobrado, braços soltos |
| `fly` | voo: punho direito à frente, esquerdo junto ao corpo |
| `flyFast` | supervelocidade: os dois punhos à frente |

Em voo o `root` inteiro gira para a cabeça apontar a direção do movimento — por
isso "braço à frente" é braço **acima da cabeça** no referencial do corpo (x = −π).

**Capa** (`cloth.js`, lógica pura testada) — PBD: integra velocidade com gravidade e
arrasto exponencial em direção à velocidade do ar, resolve restrições de distância
(estruturais, de cisalhamento e de flexão) e deriva a velocidade da posição.
7 × 12 partículas, trapézio de 0,5 → 1,0 m por 1,4 m, linha de cima presa aos ombros.
O laço das restrições é indexado e usa `Math.sqrt`: desestruturar no `for…of` e o
`Math.hypot` do V8 alocavam por restrição (~70% de tudo que o jogo alocava). As
posições vão direto no array do atributo e as normais saem de `gridNormals` (a mesma
conta do `computeVertexNormals`, sem alocar) — ver [[perfilar-alocacao-antes-de-cortar]].
Os vértices são **relativos** ao mesh, que fica na origem da capa, e os pinos saem do
`matrixWorld` de volta para o espaço do mundo: com posição absoluta, a capa se desfaria
longe da cidade ([[ADR-003-origem-flutuante]]).

## Parâmetros que importam

| Parâmetro | Valor | Por quê |
|---|---|---|
| Referencial da simulação | acompanha o herói, sem rotação | pinos quase parados mesmo a 300 m/s |
| Vento máximo | 42 m/s | acima disso a capa já está esticada; mais só desestabiliza |
| Subpasso | 1/120 s, até 4 por quadro | estabilidade com arrasto alto |
| Arrasto | 3 + 0,12·v (v ≤ 60) | capa mais "dura" quanto mais rápido |
| Colisão | cápsula pélvis–peito, r = 0,26 m | a capa não atravessa as costas |
| Reset | pino saltou > 30 m | teleporte não estica a capa pela cidade |

## Armadilhas

- **Simular em coordenadas do mundo não funciona em supervelocidade**: o pino anda
  5 m por quadro e as restrições não alcançam. O teste "passo grande com vento no
  teto" cobre o pior caso real.
- **Alocação no laço quente**: `seg.clone()` dentro do `collide` custava ~1.700
  vetores por quadro. Vetores de trabalho vivem no módulo.
- **Pose "mãos na cintura" com cotovelo no plano frontal**: fisicamente errado,
  visualmente certo; flexionar em x apontava o antebraço para a frente.
- **O emblema é uma divisa, não a letra** (ADR-002). A primeira versão, um traço em
  curva, lia como "2".

## PRs

- [[2026-09-28-pr-004-heroi-e-capa]]
- [[2026-09-28-pr-020-alocacoes-loop-quente]] — capa sem alocar por quadro
- [[2026-09-28-pr-023-origem-flutuante]] — vértices relativos ao mesh
