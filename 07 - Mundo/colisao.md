---
tipo: sistema
data: 2026-09-28
area: mundo
arquivos: [src/world/collision.js]
tags: [sistema, mundo, colisao]
---

# Colisão

## O que faz

Responde três perguntas sobre o mundo, sem three e sem alocar no caminho quente:

| Função | Pergunta | Quem usa |
|---|---|---|
| `resolveSphere(p, r, outNormal, onContact?)` | O herói (esfera) está dentro de algo? Empurra para fora; `onContact` recebe cada normal. | voo |
| `raycast(o, dir, max, out)` | Onde um raio acerta primeiro (prédio ou chão)? | visão de calor, câmera |
| `heightAt(x, z)` | Qual o telhado mais alto sob este ponto? | pouso, missões |

## Como funciona

- **Hash espacial 2D** (células de 64 m em x/z) com a lista de AABBs de cada célula.
  Um carimbo (`stamp`) por consulta evita testar a mesma caixa duas vezes.
- `resolveSphere`: ponto mais próximo da caixa → empurra ao longo da diferença. Se o
  centro já está **dentro** (túnel por velocidade alta), sai pela face mais próxima.
  O piso (`floor`: no jogo, o nível do mar, −1,2) segura a esfera em `floor + r`; a
  laje da ilha (topo em 0) e o gramado do parque (0,6) são caixas como os prédios.
- `raycast`: marcha pelo segmento em passos de meia célula visitando a vizinhança
  3×3, teste de slab por caixa; o piso entra como plano em `y = floor`.

## Armadilhas

- A esfera **não pode andar mais que o próprio raio por passo** sem risco de túnel
  em parede fina; o voo em supervelocidade precisa subdividir o passo.
- **Soma de normais não serve para cortar velocidade**: chão (0,1,0) + parede (1,0,0)
  normalizados dão a diagonal, e o corte vira subida. Quem corta velocidade usa
  `onContact`, uma normal por vez ([[2026-09-28-pr-017-voo-colisao-e-camera]]).
- **Chão da cena e da colisão têm de ser o mesmo**: o plano fixo em y = 0 enterrava as
  pernas 0,6 m no gramado e pousava o herói 1,2 m acima do mar. As alturas moram em
  `layout.js` (`QUAY`, `WATER_Y`, `LAWN`) e valem para a cena e para `collisionBoxes`
  ([[2026-09-28-pr-018-mundo-chao-e-pedestres]]).
- **O raycast ignora a caixa que contém a origem** (teste de slab): o raio tem de nascer
  fora dos prédios. A câmera traça do herói, não do ombro.

## PRs

- [[2026-09-28-pr-003-cidade-procedural]]
- [[2026-09-28-pr-017-voo-colisao-e-camera]] — normal por contato (`onContact`)
- [[2026-09-28-pr-018-mundo-chao-e-pedestres]] — piso no nível do mar; laje e gramado como caixas
