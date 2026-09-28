---
tipo: moc
data: 2026-09-28
tags: [moc, roadmap]
---

# Roadmap

Um PR por peça, com squash and merge. Marque ✅ no mesmo commit do cofre que
documenta o PR.

| # | Peça | Estado |
|---|---|---|
| 1 | Fundação: Vite + Three.js, CLAUDE.md, CI, PROJETOS.md | ✅ [[2026-09-28-pr-001-fundacao]] |
| 2 | Render: renderer, céu, luz, sombras, pós | ✅ [[2026-09-28-pr-002-pipeline-de-render]] |
| 3 | Mundo: layout procedural e prédios instanciados | ✅ [[2026-09-28-pr-003-cidade-procedural]] |
| 4 | Herói: modelo procedural + capa simulada | ✅ [[2026-09-28-pr-004-heroi-e-capa]] |
| 5 | Voo: física, input, câmera, colisão | ✅ [[2026-09-28-pr-005-voo-e-camera]] |
| 6 | HUD e minimapa | ✅ [[2026-09-28-pr-007-hud-e-minimapa]] |
| 7 | Poderes: visão de calor | ✅ [[2026-09-28-pr-008-visao-de-calor]] |
| 8 | Tráfego e pedestres | ✅ [[2026-09-28-pr-009-trafego-e-pedestres]] |
| 9 | Missões: treino, resgate e drones | ✅ [[2026-09-28-pr-010-missoes]] |
| 10 | Deploy no GitHub Pages (antecipado) | ✅ [[2026-09-28-pr-006-deploy-pages]] |

## Fase 2 — destruição e espaço

Pedido de 28/09: quebrar prédios ao bater, voar até o Sol (onde o herói fica muito mais
forte), ver a Terra e os planetas, e a visão de calor carregada de Sol atravessar a Terra.
Escala real, com velocidade proporcional à distância; a carga solar dura minutos.

| # | Peça | Estado |
|---|---|---|
| 11 | Atravessar prédios: furos, entulho, poeira | ✅ [[2026-09-28-pr-021-atravessar-predios]] |
| 12 | Desabamento: dano grande derruba a parte de cima | ✅ [[2026-09-28-pr-022-desabamento]] |
| 13a | Origem flutuante (infra para o espaço) | ✅ [[2026-09-28-pr-023-origem-flutuante]] |
| 13 | Subir ao espaço e ver a Terra (globo procedural) | ✅ [[2026-09-28-pr-024-espaco-e-terra]] |
| 14 | Sistema solar: Sol, planetas e marcadores | ✅ [[2026-09-28-pr-025-sistema-solar]] |
| 15 | Carga solar: o herói fica mais forte perto do Sol | ✅ [[2026-09-28-pr-026-carga-solar]] |
| 16 | Visão de calor carregada atravessa a Terra | ✅ [[2026-09-28-pr-027-visao-atravessa-terra]] |

Fase 2 concluída em 28/09/2026, nos PRs #21 a #27.

## Fase 3 — destruição realista

Pedido de 28/09: prédio cheio por dentro ao atravessar, explosão de verdade, prédios caindo
de forma realista, e a visão de calor cortando e quebrando prédios. O jogador escolheu:
supersônico (ou com carga solar) derruba a parte de cima de qualquer prédio, e mais devagar
cedem só os andares em volta do furo; a explosão tem clarão, fogo e fumaça.

| # | Peça | Estado |
|---|---|---|
| 17 | Interior dos prédios: andares, pilares, salas, móveis; furo aberto de verdade | ✅ [[2026-09-28-pr-028-interior-dos-predios]] |
| 18 | Explosão no impacto: clarão, bola de fogo, fumaça, onda de choque | ✅ [[2026-09-28-pr-029-explosao]] |
| 19 | Queda realista: rápido derruba, devagar cede em volta; tombar, partir no ar, nuvem de poeira | ⬜ |
| 20 | Visão de calor corta: rasgo em brasa, e o corte de lado a lado derruba | ⬜ |
