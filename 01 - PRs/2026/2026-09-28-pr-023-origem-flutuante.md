---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 23
url: https://github.com/VictorNascimento14/Superman/pull/23
branch: refactor/origem-flutuante
tags: [pr, render, espaco, precisao]
status: merged
---

# PR #23 — Origem flutuante

## 🎯 Contexto

Infraestrutura para a peça 13 do [[roadmap]] (subir ao espaço). Em escala real, o herói vai
a 10¹¹ m da cidade, e a GPU, em float32, não guarda posição absoluta com essa magnitude.
Decisão registrada em [[ADR-003-origem-flutuante]].

## 🔧 Mudanças

- `src/main.js` — grupo `world` (a origem flutuante), com a câmera como filha. Os módulos
  recebem o grupo no lugar da cena. A origem é zero a menos de 20 km da cidade e, além
  disso, acompanha o herói. É atualizada logo depois do voo.
- `src/render/sky.js` — `createSky(scene, renderer, preset, world)`: céu, luzes e estrelas
  vão no grupo; neblina e mapa de ambiente, na `Scene`.
- `src/player/camera.js` e o modo de depuração — `lookAt` com o alvo convertido para espaço
  de render.
- `src/powers/heatvision.js` — o olho volta ao espaço do mundo e a mira vai para o de
  render no `lookAt` do feixe.
- `src/player/hero.js` — os pinos da capa saem do `matrixWorld` para o espaço do mundo, e os
  vértices ficam relativos ao mesh, que vai para a origem da capa.
- Testes: a câmera com o mundo deslocado a 10⁹ m sai com posição e orientação de render
  idênticas às de perto da cidade. O e2e ganha o cenário **origem flutuante**.

## 🧠 Decisões técnicas

- **Um grupo pai, e não subtrair a origem em cada módulo.** O three multiplica as matrizes
  em float64, então a diferença sai exata. Os módulos continuam em coordenadas verdadeiras
  (colisão, voo e missões não mudam) e só convertem onde misturam espaços.
- **Origem zero perto da cidade**: nenhuma mudança no jogo de sempre, que o e2e confirma.
- **Vértice relativo na capa**: posição absoluta num `Float32Array` perderia precisão longe,
  mesmo com a origem flutuante.

## ⚠️ Armadilhas e aprendizados

- **Neblina e ambiente num grupo não valem.** Passar o grupo ao `createSky` deixou a cidade
  escura. Pegamos isso comparando screenshots de antes e depois.
- A conversão precisa da matriz do grupo **deste** quadro. A origem é atualizada antes da
  capa, da câmera e da visão de calor.
- O véu branco longe da cidade é o bloom do céu Preetham abaixo do horizonte, onde não há
  chão. Já era assim antes deste PR, e a Terra da peça 13 cobre esse vazio.

## 🧪 Como testar

1. `npm test` (68) e `npm run build`. `npm run e2e`: missões com 922 pontos, atravessar,
   desabar, e origem flutuante com a câmera a 5,4 m da origem de render, a 5·10⁸ m da
   cidade.
2. No jogo, nada muda na cidade.

## 📎 Documentação afetada

- [[ADR-003-origem-flutuante]] (nova)
- [[pipeline-de-render]], [[voo-e-camera]], [[visao-de-calor]], [[heroi-e-capa]],
  [[runbook-e2e]]
- [[roadmap]], [[2026]] (changelog)
