---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 15
url: https://github.com/VictorNascimento14/Superman/pull/15
branch: fix/missoes-borda-e-drones
tags: [pr, missoes, e2e]
status: merged
---

# PR #15 — Missões: anéis longe da ilha, drones acima dos prédios e e2e por tipo

## 🎯 Contexto

Achados da revisão do [[2026-09-28-pr-014-revisao-de-codigo]] no sistema de missões. O mais
grave congelava o jogo.

## 🔧 Mudanças

- `src/game/missionLogic.js` — se a partida não comporta nenhum anel à frente (sobre o mar,
  ou na beirada olhando para fora), `makeRingCourse` recomeça de dentro da ilha, rumo ao
  centro. Antes devolvia `[]`, e `ringCrossed(…, undefined)` lançava TypeError **a cada
  quadro** dentro do `setAnimationLoop`: a imagem congelava até alguém apertar N. A
  colocação dos anéis foi para `placeRings`.
- `src/game/missions.js`:
  - os drones orbitam 20 m acima do telhado mais alto num raio de 40 m (amostrado em 16
    direções × 3 raios);
  - o dano acende o drone: `emissiveIntensity` nascia 0 e nada o subia;
  - os marcadores dos drones não ficam no minimapa quando a missão acaba no quadro. O
    `end()` já tinha posto os de base, mas o `update()` sobrescrevia com a lista velha;
  - `doneBy` conta as vitórias por tipo.
- `scripts/e2e/run.mjs` — só passa com uma vitória de **cada** tipo.
- `tests/missions.test.js` — circuito a partir da beirada e do mar (3 partidas × 20
  sementes).

## 🧠 Decisões técnicas

- **Recomeçar dentro da ilha em vez de proteger o update.** Um `if (!r)` no
  `updateRings` evitaria o crash, mas deixaria uma missão impossível. A correção vive na
  lógica pura, onde o teste pega.
- **Recomeçar a 260 m da borda útil (`HALF − 60 − 260`)**: como o passo máximo é 260 m,
  qualquer primeira curva cabe. Em 2.000 partidas de fora, o pior caso saiu com 5 anéis e
  98,7% com os 8.
- **A sequência de sorteios não muda quando a partida é boa**, então o e2e (que parte da
  ilha) gera os mesmos anéis.
- **Drones acima dos telhados em vez de órbita menor.** Limitar o raio a 12 m (dentro do
  cruzamento) também resolveria, mas amontoaria os 5 drones.

## Medição

Drones: 2.000 missões simuladas em cruzamentos reais, com o raio do drone de 2,2 m.

| | Órbita dentro de prédio | Missões com drone > 25% da órbita dentro |
|---|---|---|
| Antes | 7,9% | 533 (27%) |
| Depois | 0,0% | 0 |

A altura máxima de órbita foi de 85 m para 279 m, sobre o centro.

## ⚠️ Armadilhas e aprendizados

- Uma exceção dentro do `setAnimationLoop` não derruba o loop: ele continua sendo chamado
  e lança de novo a cada quadro. A tela congela sem nenhum erro visível para o jogador.
- O e2e contava vitórias, não tipos: "3 missões cumpridas" passava com anéis → resgate ✗
  → drones → anéis.

## 🧪 Como testar

1. `npm test` (43), `npm run build` e `npm run e2e` (uma vitória de cada tipo: 79 s, 921 pontos).
2. `npm run dev`: voe para o mar e espere a missão de anéis (1ª, 4ª, 7ª…). O circuito
   aparece na ilha, sem congelar.

## 📎 Documentação afetada

- [[missoes]], [[runbook-e2e]]
- [[2026]] (changelog)
