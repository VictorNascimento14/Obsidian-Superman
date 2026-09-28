---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 11
url: https://github.com/VictorNascimento14/Superman/pull/11
branch: chore/e2e-autopiloto
tags: [pr, infra, teste]
status: merged
---

# PR #11 — Autopiloto de ponta a ponta versionado

## 🎯 Contexto

No [[2026-09-28-pr-010-missoes]] as missões foram provadas cumpríveis por um
autopiloto que vivia fora do repositório. Sem versionar, a prova morre com a sessão.

## 🔧 Mudanças

- `scripts/e2e/autopilot.js` — o jogador automático (roda na página).
- `scripts/e2e/run.mjs` — build, preview, Chrome headless, asserções, screenshots.
- `package.json` — `npm run e2e`; `puppeteer-core` em devDependencies (não baixa Chrome).
- `src/game/missions.js` — getter `done`.
- `CLAUDE.md` — quando rodar o e2e.
- `.gitignore` — `e2e-out/`.

## 🧠 Decisões técnicas

- **`puppeteer-core`, não `puppeteer`**: usa o Chrome da máquina; o pacote completo
  baixaria ~170 MB de Chromium a cada `npm ci`.
- **Fora do CI**: o runner não tem GPU; com SwiftShader a cidade roda a poucos fps e o
  autopiloto perderia a janela das missões. Fica como gate local, declarado no CLAUDE.md.
- **Roda sobre o `dist`**, não o dev server: testa o que vai para o Pages.

## 🧪 Como testar

`npm run e2e` → "OK — 3 missões cumpridas" (81 s, 921 pontos em 28/09).

## 📎 Documentação afetada

- [[runbook-e2e]]
- [[2026]] (changelog)
