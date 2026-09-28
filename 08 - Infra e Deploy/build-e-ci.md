---
tipo: sistema
data: 2026-09-28
area: infra
arquivos: [package.json, vite.config.js, .github/workflows/ci.yml, .github/workflows/pr-documentacao.yml]
tags: [sistema, infra, ci]
---

# Build e CI

## O que faz

- `npm run dev` — Vite em `127.0.0.1:5173`.
- `npm test` — `node --test tests/`, só lógica pura (sem DOM, sem WebGL).
- `npm run build` — bundle em `dist/`, `base: './'` (caminhos relativos).

## Workflows

| Workflow | Quando | O que exige |
|---|---|---|
| `ci.yml` | todo PR e push na `main` | `npm ci`, `npm test`, `npm run build` |
| `pr-documentacao.yml` | PR aberto/editado | seção `## 📓 Documentação` com link para o cofre |

## Armadilhas

- O check de documentação lê o corpo do PR **no evento**. Se o PR nasce sem a seção
  e ela entra por edição, o evento `edited` roda o check de novo — não precisa de push.

## PRs

- [[2026-09-28-pr-001-fundacao]]
