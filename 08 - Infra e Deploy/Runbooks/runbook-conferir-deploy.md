---
tipo: runbook
data: 2026-09-28
tags: [runbook, deploy]
---

# Runbook — conferir o deploy do Pages

1. Último run: `gh run list -R VictorNascimento14/Superman -w "Deploy no GitHub Pages" -L 1`.
2. Se falhou: `gh run view <id> --log-failed | tail -40`. Teste ou build vermelho
   bloqueia o deploy — o site continua na versão anterior.
3. Conferir o que está no ar: `curl -s https://victornascimento14.github.io/Superman/ | grep -o 'assets/index-[^"]*'`
   e comparar com o `dist/` do build local da mesma `main`.
4. Re-disparar sem commit: `gh workflow run pages.yml -R VictorNascimento14/Superman`.
