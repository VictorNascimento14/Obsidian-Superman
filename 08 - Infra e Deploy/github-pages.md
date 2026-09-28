---
tipo: sistema
data: 2026-09-28
area: infra
arquivos: [.github/workflows/pages.yml, vite.config.js]
tags: [sistema, infra, deploy]
---

# Deploy no GitHub Pages

**URL:** https://victornascimento14.github.io/Superman/

## Como funciona

- `pages.yml` roda em todo push na `main` (ou seja, em todo squash and merge):
  `npm ci` → `npm test` → `npm run build` → `upload-pages-artifact(dist)` →
  `deploy-pages`.
- O Pages foi habilitado com `build_type=workflow` (fonte = GitHub Actions, sem
  branch `gh-pages`).
- `vite.config.js` usa `base: './'`: o mesmo build serve em `/` no dev e em
  `/Superman/` no Pages, sem configurar caminho.

## Conferir um deploy

Ver [[runbook-conferir-deploy]].

## PRs

- [[2026-09-28-pr-006-deploy-pages]]
