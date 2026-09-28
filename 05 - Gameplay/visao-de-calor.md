---
tipo: sistema
data: 2026-09-28
area: gameplay
arquivos: [src/powers/heatvision.js, src/powers/energy.js, src/player/camera.js]
tags: [sistema, poderes, visao-de-calor]
---

# Visão de calor

![[visao-de-calor.jpg]]

**Botão direito do mouse** ou **F**, segurado.

## Como funciona

1. **Mira**: raio do centro da câmera (a mira do HUD), alcance 700 m, contra a
   [[colisao]] (prédio ou chão) e contra os **alvos registrados** em `targets`
   (esferas `{ pos, radius, hit(dt, ponto) }`). O mais próximo vence.
2. **Feixes**: um por olho, do olho até o ponto mirado. Cada feixe é um cilindro
   aberto com núcleo branco-quente fino e halo vermelho largo, **aditivos e em HDR**
   (cor > 1): passam do limiar 1.0 do bloom do [[pipeline-de-render]]. Tremor leve na
   espessura para não parecer laser.
3. **Impacto**: luz pontual laranja, sprite de brilho, fagulhas com gravidade (pool de
   400) e **marcas de queimado** (400 quads instanciados em anel, com polygon offset)
   a cada 40 ms.
4. **Olhos** acendem (emissivo do material dos olhos do [[heroi-e-capa]]).

## Energia (`energy.js`, testada)

| Parâmetro | Valor |
|---|---|
| Dreno | 22%/s (≈ 4,5 s de disparo contínuo) |
| Recarga | 18%/s, depois de 0,8 s sem disparar |
| Trava | esvaziou → só religa com 20% |

A trava existe para o zero não virar pisca-pisca (liga, drena, desliga, recarrega um
quadro, liga…).

## Carregada de Sol

Com a [[carga-solar]], o alcance vai a `700 m × (1 + 5 × carga)` (4,2 km cheia), o dano nos
alvos a `× (1 + 4 × carga)`, e a reserva drena `× (1 − carga)` — cheia, o Sol paga o disparo
inteiro (`energy.update(dt, wants, drainMul)`). O raio engrossa até 2,2 vezes e passa de
laranja para branco-dourado. Disparar gasta a carga mais depressa (`SOLAR.beam`).

Com 35% de carga ou mais, o raio que acerta a Terra a atravessa e deixa um buraco na entrada
e na saída — ver [[atravessar-a-terra]]. Para isso o `update(dt, wants, over)` aceita a
direção da mira assistida (`over`) no lugar da direção da câmera.

## Corta e quebra prédios

![[visao-corta.jpg]]

**Corte** (`powers/cuts.js`, lógica pura testada):

- a cada quadro com o raio numa fachada, as células de 1 m em volta do ponto (0,9 m de cada
  lado, ×3 com a carga solar cheia) ficam queimadas, por prédio, face e faixa de 8 m de altura;
- perto da divisa de duas faixas (2,5 m), o raio queima as duas, porque a mira sobe e desce ao
  varrer a parede;
- quando 70% da largura da face está queimada numa faixa, o prédio está **fatiado**: a parte
  de cima cai a partir da altura do corte (`collapses.slice`). É um corte limpo: não esmaga os
  andares de baixo, só tomba sobre a borda do corte, escorrega para fora (para onde o raio ia)
  e cai. O toco fica na altura do corte, com escombros em cima (aviso **CORTADO!**);
- **parado num ponto** por 0,6 s (0,15 s com a carga solar), o raio estoura um furo. Ele entra
  no jogo no quadro seguinte como uma ruptura, a 120 m/s: furo aberto, explosão, interior e
  andares cedendo. Dois furos na mesma faixa derrubam um prédio de ~40 m.

**Brasa** (`heatvision.js`): cada marca de queimado ganha um brilho aditivo em HDR que esfria
em 2,5 s. Com o raio varrendo, vira uma linha incandescente que escurece. `surfaceHit` expõe o
ponto, a normal e a caixa acertados no quadro.

## Câmera sobre o ombro

Com a câmera centrada, o ponto mirado fica **exatamente atrás do herói**: o corpo
esconde os feixes. A câmera agora fica 1,1 m à direita (1,7 m ao mirar, e 30% mais
perto), e centraliza entre 20 e 80 m/s — em voo rápido o ombro não faz sentido.

## Origem flutuante

O olho (`getWorldPosition`) sai em espaço de render e a mira em espaço verdadeiro: o olho volta
para o do mundo (`scene.worldToLocal`), e o `lookAt` do feixe recebe a mira convertida —
ver [[ADR-003-origem-flutuante]].

## PRs

- [[2026-09-28-pr-008-visao-de-calor]]
- [[2026-09-28-pr-023-origem-flutuante]] — olho e mira no mesmo espaço
- [[2026-09-28-pr-026-carga-solar]] — alcance, dano e raio com a carga solar; o Sol paga a reserva
- [[2026-09-28-pr-027-visao-atravessa-terra]] — a mira assistida guia o raio até a Terra
- [[2026-09-28-pr-031-visao-corta]] — corta (fatia) e quebra (estoura furo) prédios; corte em brasa
