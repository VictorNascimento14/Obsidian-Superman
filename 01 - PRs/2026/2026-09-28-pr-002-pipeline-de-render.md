---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 2
url: https://github.com/VictorNascimento14/Superman/pull/2
branch: feat/render-pipeline
tags: [pr, render]
status: merged
---

# PR #2 — Pipeline de render: céu, sombras e pós

## 🎯 Contexto

Antes da cidade, a luz. Todo sistema visual seguinte (fachadas, vidro, janelas
acesas à noite, visão de calor) depende de céu, ambiente, sombra e bloom estarem certos.

## 🔧 Mudanças

- `src/render/renderer.js`, `sky.js`, `post.js`, `quality.js` — ver [[pipeline-de-render]].
- `src/main.js` — palco provisório (chão + 12 torres em anel) com câmera orbitando,
  tecla **T** troca o horário, gancho `window.__game` para teste headless.
- `index.html` — favicon vazio (o 404 do favicon sujava o console).

## 🧠 Decisões técnicas

- **ACES em vez de AgX**: AgX deixou o dia lavado e acinzentado nas screenshots.
- **Bloom com limiar 1.0**: com 0.85 o reflexo do céu nos vidros florescia de dia.
- **Cúpula própria à noite** em vez de forçar o `Sky` com elevação negativa (preto).

## ⚠️ Armadilhas e aprendizados

- Três avisos do three r186 corrigidos: `PCFSoftShadowMap` removido, `Clock` depreciado,
  `renderer.info` medindo só o último pass do pós.

## 🧪 Como testar

1. `npm run dev` → torres com sombra, céu com nuvens.
2. **T** quatro vezes: amanhecer, dia, entardecer (sol laranja atrás das torres), noite (estrelas).
3. `?q=baixa` e `?q=alta` na URL.

Verificado por screenshot headless (Chrome + puppeteer-core) nos quatro horários, console limpo.

## 📎 Documentação afetada

- [[pipeline-de-render]]
- [[2026]] (changelog)
