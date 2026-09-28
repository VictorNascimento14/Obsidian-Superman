---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 13
url: https://github.com/VictorNascimento14/Superman/pull/13
branch: perf/recorte-e-agua
tags: [pr, desempenho, mundo]
status: merged
---

# PR #13 — Recorte do tráfego por distância e água mais natural

## 🎯 Contexto

O [[2026-09-28-pr-009-trafego-e-pedestres]] quase dobrou os triângulos por quadro
(~440 mil → ~800 mil) e ficou anotado como próximo alvo. A água mostrava um padrão de
pontos regular desde o [[2026-09-28-pr-007-hud-e-minimapa]].

## 🔧 Mudanças

- `src/world/trafficView.js` — só carros a ≤ 900 m e pedestres a ≤ 300 m da câmera
  vão para a GPU, compactados no começo do buffer (matriz e cor). `count` = visíveis.
- `src/main.js` — passa a posição da câmera.
- `src/world/textures.js`, `city.js` — normal map da água com 14 ondas em direções
  variadas (512 px), repetição 160 → 70, relevo mais baixo.

## Medição

| Cena | Triângulos antes | Depois |
|---|---|---|
| Rua (14 m) | 806 mil | 575 mil |
| Alto (380 m) | 782 mil | 520 mil |
| Noite (380 m) | 773 mil | 509 mil |

**FPS: inconclusivo** — ver [[fps-headless-nao-mede-ganho]].

![[agua-antes-depois.jpg]]

## 🧪 Como testar

1. `npm test` (42) e `npm run e2e` (3 missões, 923 pontos).
2. `npm run dev`: descer numa avenida — carros e pedestres continuam lá; subir — eles
   somem longe, onde não se veriam. Voar sobre o mar: água sem padrão de bolinhas.

## 📎 Documentação afetada

- [[trafego-e-pedestres]]
- [[fps-headless-nao-mede-ganho]]
- [[2026]] (changelog)
