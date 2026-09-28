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
| `resolveSphere(p, r, outNormal)` | O herói (esfera) está dentro de algo? Empurra para fora. | voo |
| `raycast(o, dir, max, out)` | Onde um raio acerta primeiro (prédio ou chão)? | visão de calor, câmera |
| `heightAt(x, z)` | Qual o telhado mais alto sob este ponto? | pouso, missões |

## Como funciona

- **Hash espacial 2D** (células de 64 m em x/z) com a lista de AABBs de cada célula.
  Um carimbo (`stamp`) por consulta evita testar a mesma caixa duas vezes.
- `resolveSphere`: ponto mais próximo da caixa → empurra ao longo da diferença. Se o
  centro já está **dentro** (túnel por velocidade alta), sai pela face mais próxima.
  O chão (y = 0) segura a esfera em `y = r`.
- `raycast`: marcha pelo segmento em passos de meia célula visitando a vizinhança
  3×3, teste de slab por caixa; o chão entra como plano.

## Armadilhas

- A esfera **não pode andar mais que o próprio raio por passo** sem risco de túnel
  em parede fina; o voo em supervelocidade precisa subdividir o passo.

## PRs

- [[2026-09-28-pr-003-cidade-procedural]]
