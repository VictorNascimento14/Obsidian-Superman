---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 19
url: https://github.com/VictorNascimento14/Superman/pull/19
branch: fix/render-sombras
tags: [pr, render, sombras]
status: merged
---

# PR #19 — Render: sombras estáveis com o sol baixo

## 🎯 Contexto

Achados da revisão do [[2026-09-28-pr-014-revisao-de-codigo]] na fatia de render. A luz de
sombra ficava perto demais do foco quando o sol estava baixo, o snap de texel era feito nos
eixos errados, e o `?q=` aceitava chaves herdadas de `Object`.

## 🔧 Mudanças

- `src/render/sky.js`:
  - a luz de sombra fica a 2.000 m do foco (`LIGHT_DIST`), com `far` 3.000 e bias
    −0,0002;
  - o centro da caixa de sombra é arredondado para a grade de texel nos eixos `right` e
    `up` da câmera de sombra, calculados quando o horário muda.
- `src/render/quality.js` — `Object.hasOwn(QUALITY, q)`.
- `tests/quality.test.js` — `?q=constructor|toString|__proto__` cai no padrão.

## 🧠 Decisões técnicas

- **Afastar a luz em vez de encolher a caixa.** Com o sol a 18° (o mínimo, que vale para
  amanhecer, entardecer e noite), os ±260 m da caixa cobrem ~840 m de chão na direção do
  sol. A 800 m, prédios altos desse lado ficavam antes do `near` e não entravam no shadow
  map: a sombra longa deles sumia, e aparecia ou desaparecia conforme a caixa seguia o
  herói. Encolher a caixa encurtaria o alcance da sombra em toda hora do dia.
- **Bias recalculado, não mantido.** Em câmera ortográfica o bias é em profundidade
  normalizada: −0,0004 × 1.590 m ≈ −0,0002 × 2.990 m ≈ 0,6 m no mundo.
- **Snap na base da luz.** Projetar o foco em `right` e `up`, arredondar e reconstruir. A
  componente ao longo do sol fica livre, porque não muda a projeção.

## Medição

Com as matrizes reais do three no Node:

| | Antes | Depois |
|---|---|---|
| Caixa de sombra fora do `[near, far]`, sol a 18° (chão e telhados até 250 m) | 20,9% | 0% |
| Idem, sol a 30° | 5,9% | 0% |
| Variação da fração de texel de um ponto fixo, com o herói andando (dia) | 0,97 | 0,000 |
| Idem, entardecer | 0,98 × 0,90 | 0,000 |

Visual, no headless, mesma cena antes e depois: **sem regressão**, com 0,05% dos pixels
mudando no entardecer e 0,07% de dia. Sem acne e sem sombra descolada. Na cena testada, a
área onde cai a sombra longa do Planeta Diário já estava na sombra de prédios próximos (com
o sol a 18°, toda sombra tem ~3× a altura do prédio), então o print não mostra o ganho. O
ganho fica na conta acima.

## ⚠️ Armadilhas e aprendizados

- A nota do pipeline dizia "arredondada para a grade de texel", e estava: só que nos eixos
  errados. A grade do shadow map segue o sol, não o mundo.
- `QUALITY[q]` com `q` vindo da URL herda de `Object.prototype`.

## 🧪 Como testar

1. `npm test` (47), `npm run build` e `npm run e2e` (anéis 1 · resgate 1 · drones 1, 923 pontos). O e2e não é exigido para
   render, mas rodou.
2. `npm run dev`: T até o entardecer, voe baixo em linha reta e observe as bordas das
   sombras, que não nadam mais. `?q=constructor` abre no preset médio.

## 📎 Documentação afetada

- [[pipeline-de-render]]
- [[2026]] (changelog)
