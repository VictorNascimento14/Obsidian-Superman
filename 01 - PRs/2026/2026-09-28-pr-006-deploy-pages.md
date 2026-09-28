---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 6
url: https://github.com/VictorNascimento14/Superman/pull/6
branch: chore/deploy-pages
tags: [pr, infra, deploy]
status: merged
---

# PR #6 — Publicar o jogo no GitHub Pages

## 🎯 Contexto

Com o voo pronto (#5), o jogo já é jogável. Publicar agora faz cada PR seguinte
chegar a um link, sem ninguém precisar clonar.

## 🔧 Mudanças

- `.github/workflows/pages.yml` — build + teste + deploy do `dist/`. Ver [[github-pages]].
- `README.md` — link jogável e resumo dos controles.
- Pages habilitado por API (`POST /repos/.../pages`, `build_type=workflow`).

## 🧠 Decisões técnicas

- **Deploy saído da `main`**, não de tag: o repositório segue squash and merge com CI
  verde, então a `main` já é a versão boa.
- **Testes rodam de novo no deploy**: barato, e impede publicar algo que só passou
  no PR por causa de outro PR mergeado no meio.

## 🧪 Como testar

1. Depois do merge: [[runbook-conferir-deploy]].
2. Abrir https://victornascimento14.github.io/Superman/ e voar.

## 📎 Documentação afetada

- [[github-pages]]
- [[runbook-conferir-deploy]]
- [[2026]] (changelog)
