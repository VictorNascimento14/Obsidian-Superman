---
tipo: runbook
data: 2026-09-28
tags: [runbook, teste, e2e]
---

# Runbook — rodar o teste de ponta a ponta

```bash
npm run e2e
```

1. Builda (`vite build`) e serve o `dist/` em `127.0.0.1:4174` (`vite preview`).
2. Abre o Chrome headless (`/usr/bin/google-chrome`, ou `CHROME_PATH`).
3. Injeta `scripts/e2e/autopilot.js`, que joga as missões como um jogador —
   ver [[verificacao-ponta-a-ponta-por-autopiloto]].
4. Imprime a cada 2 s: missão atual, cumpridas, pontos. Salva uma screenshot por
   missão em `e2e-out/`.
5. **Passa** com 3 missões cumpridas e console sem erro; **falha** (exit 1) no
   contrário ou depois de 240 s.

Referência de 28/09/2026: 3 missões em **81 s**, 921 pontos.

## Quando rodar

Antes de publicar PR que toque voo, câmera, colisão, visão de calor ou missões.
Não roda no CI (precisa de Chrome com GPU).

## Se falhar

- **Travou numa missão**: abra a screenshot dela em `e2e-out/`. Se o jogo parece
  certo e o autopiloto é que não sabe resolver a situação nova, melhore o autopiloto
  **e diga isso no PR** — mudar o teste para passar exige justificar.
- **Erro no console**: é bug. O e2e não tolera `pageerror`.
