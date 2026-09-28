---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 4
url: https://github.com/VictorNascimento14/Superman/pull/4
branch: feat/heroi-e-capa
tags: [pr, gameplay, heroi]
status: merged
---

# PR #4 — Herói procedural com poses e capa simulada

## 🎯 Contexto

Sem Blender na máquina e sem asset baixado (ADR-001), o herói é montado com
primitivas. A capa é o que mais vende a sensação de voo, então ganhou simulação de
verdade, não animação pronta.

## 🔧 Mudanças

- `src/player/hero.js` — rig, materiais, emblema em canvas, poses e capa. Ver [[heroi-e-capa]].
- `src/player/cloth.js` — PBD puro.
- `tests/cloth.test.js` — 4 testes: caimento, vento de frente, pior caso de passo, colisão.
- `src/main.js` — vitrine provisória: **P** troca a pose; em voo o herói circula.
  Gancho `camHero` para screenshot relativa ao herói.

## 🧠 Decisões técnicas

- **Capa no referencial do herói + vento com teto**: a única forma estável em supervelocidade.
- **Poses por média ponderada de Euler** em vez de `AnimationMixer`: são poucas juntas
  e ângulos pequenos entre poses; sem clips para carregar.

## ⚠️ Armadilhas e aprendizados

Ver [[heroi-e-capa]]: simulação no mundo explode, alocação no `collide`, eixo do
cotovelo na pose akimbo, emblema que parecia "2".

## 🧪 Como testar

1. `npm test` — 20 testes.
2. `npm run dev`, **P**: idle → hover → fly → flyFast; a capa reage à velocidade.

## 📎 Documentação afetada

- [[heroi-e-capa]]
- [[2026]] (changelog)
