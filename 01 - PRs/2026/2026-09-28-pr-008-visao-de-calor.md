---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 8
url: https://github.com/VictorNascimento14/Superman/pull/8
branch: feat/visao-de-calor
tags: [pr, gameplay, poderes]
status: merged
---

# PR #8 — Visão de calor

## 🎯 Contexto

O segundo poder do herói e a ferramenta das missões de combate: os drones do
próximo PR se registram como alvo aqui.

## 🔧 Mudanças

- `src/powers/heatvision.js` — mira, feixes, fagulhas, queimado, luz. Ver [[visao-de-calor]].
- `src/powers/energy.js` + `tests/energy.test.js` — energia com trava e raio-esfera (3 testes).
- `src/player/camera.js` — câmera sobre o ombro.
- `src/core/input.js` — `isDown(code)` para tecla segurada.
- `src/main.js` — botão direito/F dispara; barra de energia no HUD.

## 🧠 Decisões técnicas

- **Feixe em HDR aditivo** em vez de linha/shader próprio: o bloom já existente faz o brilho.
- **Uma `PointLight` sempre na cena, com intensidade 0 parada**: ligar/desligar luz
  muda o número de luzes e força recompilar todos os shaders (engasgo).
- **Alvos como esferas num `Set`** — a missão registra e remove; a visão de calor não
  conhece drone.

## ⚠️ Armadilhas e aprendizados

- **O herói escondia os próprios feixes**: câmera centrada põe a mira atrás do corpo.
  Resolvido com câmera sobre o ombro.
- **Fagulhas como quadrados gigantes à noite**: `PointsMaterial` sem textura desenha
  quadrado, e cor 5× estourava no bloom. Textura radial e cor 2,4×.

## 🧪 Como testar

1. `npm test` — 32 testes.
2. `npm run dev`, voar até perto de um prédio, segurar botão direito (ou F): feixes,
   fagulhas e queimado na parede; a barra vermelha esvazia e recarrega.

## 📎 Documentação afetada

- [[visao-de-calor]]
- [[2026]] (changelog)
