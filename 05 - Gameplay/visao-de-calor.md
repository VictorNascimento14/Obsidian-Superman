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
