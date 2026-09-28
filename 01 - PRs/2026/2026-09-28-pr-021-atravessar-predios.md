---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 21
url: https://github.com/VictorNascimento14/Superman/pull/21
branch: feat/atravessar-predios
tags: [pr, destruicao, voo, colisao]
status: merged
---

# PR #21 — Atravessar prédios

## 🎯 Contexto

Primeira peça da fase 2 do [[roadmap]]: bater num prédio voando rápido deve quebrá-lo e
atravessá-lo, como aconteceria com um herói assim. O desabamento da parte de cima fica
para a peça seguinte.

## 🔧 Mudanças

- `src/world/collision.js` — o `onContact(n, idx)` recebe o índice da caixa (−1 = piso) e
  é chamado **antes** do empurrão; se devolver `true`, a caixa não empurra. `skip` lista as
  caixas que o herói está atravessando.
- `src/world/layout.js` — os níveis de prédio são `breakable`. Laje, gramado e globo não.
- `src/player/flight.js` — `SMASH`: fura com ≥ 30 m/s para dentro da parede, cobra 6 m/s
  na fachada e `0,012 × v` por metro lá dentro, com piso de 10 m/s. Eventos `breach` na
  entrada e na saída. Sem pouso enquanto estiver dentro.
- `src/fx/debrisSim.js` (novo, puro) — entulho em arrays planos: gravidade, quique, atrito,
  giro e sono.
- `src/fx/breach.js` (novo) — furo em decalque, entulho instanciado (1.200) e poeira em
  `Points` com shader próprio.
- `src/ui/hud.js`, `src/main.js` — poeira na tela com a câmera dentro de um prédio;
  tremor e som.
- `src/audio/audio.js` — `breach`: estalo na entrada, estrondo na saída.
- Testes: 4 de voo (fura e sai mais devagar; devagar é sólido; raspar não fura; lá dentro
  não pousa nem prende), 5 de entulho e 1 de layout. `scripts/e2e` ganha o cenário
  **atravessar**.

## 🧠 Decisões técnicas

- **O contato decide antes do empurrão.** A primeira versão empurrava e só depois
  avisava: a esfera ficava tangente à face, parecia "já ter saído", e o herói entrava e
  saía da mesma parede a cada subpasso.
- **O furo é desenhado, não recortado.** As paredes são cascas de face única numa malha
  mesclada por estilo. Recortar exigiria CSG ou um shader de descarte por prédio. O decalque
  com borda de concreto e vergalhão lê como rombo a qualquer distância, e a poeira na tela
  cobre a travessia por dentro.
- **Perda proporcional à velocidade** (`perMeter × v` por metro): o prédio tira a mesma
  fração em qualquer marcha. Com perda fixa, o supersônico nem sentiria o prédio, e o
  cruzeiro pararia lá dentro.
- **O entulho da saída acompanha o herói** (55–110% da velocidade dele). A 20–70%, ficava
  atrás da câmera e ninguém via.

## ⚠️ Armadilhas e aprendizados

Registradas em [[destruicao]]: o laço do empurrão, o furo dentro da parede, o pouso no
telhado por dentro e a parede × chão do entulho. No e2e, o ponto de partida do cenário
precisa estar na rua (`heightAt` = 0). O raycast só olha para frente, e começar dentro de
outro prédio deixava a câmera lá dentro.

## 🧪 Como testar

1. `npm test` (58), `npm run build` e `npm run e2e` (missões cumpridas, 921 pontos; e
   atravessar: 3 rupturas, 104 pedaços de entulho, passou).
2. `npm run dev`: acelere (Shift) contra um prédio. Você sai do outro lado com poeira na
   tela, e na fachada ficam os rombos. Devagar, o prédio continua sólido.

## 📎 Documentação afetada

- [[destruicao]] (nova), [[colisao]], [[voo-e-camera]], [[runbook-e2e]]
- [[roadmap]] (fase 2), [[arquitetura]]
- [[2026]] (changelog)
