---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 3
url: https://github.com/VictorNascimento14/Superman/pull/3
branch: feat/cidade-procedural
tags: [pr, mundo, cidade]
status: merged
---

# PR #3 — Metropolis procedural com colisão

## 🎯 Contexto

O herói precisa de onde voar. A cidade é o maior sistema do jogo e o que mais pesa
no desempenho, então nasceu mesclada e instanciada desde o início.

## 🔧 Mudanças

- `src/world/layout.js` — gerador puro: grade, densidade, lotes, estilos, recuos,
  parque, Planeta Diário, caixas de colisão. Ver [[cidade-procedural]].
- `src/world/collision.js` — hash espacial, esfera, raio, altura do telhado. Ver [[colisao]].
- `src/world/textures.js` — fachadas, chão, grama, telhado, ondas, letreiro em canvas.
- `src/world/city.js` — meshes mesclados por estilo e instanciados.
- `src/main.js` — cidade no lugar do palco provisório; sobrevoo do centro.
- `tests/layout.test.js` (7 testes) e `tests/collision.test.js` (7 testes).

## 🧠 Decisões técnicas

- **Mesclar por estilo, não instanciar por prédio**: cada prédio tem dimensões e UV
  próprios; mesclar dá 4 draw calls para todas as paredes.
- **UV em metros** + ladrilho 8×8 + deslocamento por prédio: janelas do mesmo tamanho
  em todo lugar, sem repetição visível.
- **Colisão por AABB** em vez de malha: os prédios são caixas; o custo é mínimo.

## ⚠️ Armadilhas e aprendizados

Quatro, todas registradas em [[cidade-procedural]]: cornija com tampa cobrindo o
telhado, tom de vertex color sobre textura saturada, `mergeGeometries` misturando
indexada e não indexada, ladrilho 4×4 repetindo à noite. Todas achadas por
screenshot headless — nenhuma quebraria teste.

## 🧪 Como testar

1. `npm test` — 16 testes.
2. `npm run dev` — sobrevoo do centro; **T** para a noite (cidade acesa).

## 📎 Documentação afetada

- [[cidade-procedural]]
- [[colisao]]
- [[2026]] (changelog)
