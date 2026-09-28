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
4. Imprime a cada 2 s: missão atual, vitórias por tipo (anéis · resgate · drones) e
   pontos. Salva uma screenshot por missão em `e2e-out/`.
5. **Passa** com ao menos uma vitória de **cada** tipo e console sem erro; **falha**
   (exit 1) no contrário ou depois de 240 s. Três vitórias de qualquer tipo não bastam:
   um resgate quebrado passaria com anéis de novo ([[2026-09-28-pr-015-missoes-borda-e-drones]]).
6. Depois das missões, para o autopiloto e roda o cenário **atravessar**: da rua, a
   150 m/s contra um prédio, exige entrada + saída, entulho e o herói do outro lado
   ([[2026-09-28-pr-021-atravessar-predios]]). O alvo é uma face com rua livre a **12 m**
   (a 40 m o ponto já cai no quarteirão vizinho).
7. Cenário **desabar**: supersônico num prédio estreito; em 8 s ele tem de ter caído
   inteiro, com o teto da colisão na altura do toco ([[2026-09-28-pr-022-desabamento]]).
8. Cenário **origem flutuante**: o herói a 5·10⁸ m da cidade; a câmera tem de ficar a
   menos de 100 m da origem de render ([[2026-09-28-pr-023-origem-flutuante]]). Screenshot
   em `e2e-out/longe.png`.
9. Cenário **espaço**: de 25 km, subindo na vertical com boost, tem de passar de 1.000 km
   em 8 s; descendo na vertical (C + boost), tem de voltar a menos de 5 km em 15 s
   ([[2026-09-28-pr-024-espaco-e-terra]]). Screenshot em `e2e-out/espaco.png`.
10. Cenário **sistema solar**: de 20.000 km, mirando o Sol com boost, tem de chegar a menos de
    100.000 km da superfície dele em 45 s (leva ~15 s); depois uma foto de Saturno de perto
    ([[2026-09-28-pr-025-sistema-solar]]). Screenshots em `e2e-out/sol.png` e
    `e2e-out/saturno.png`.
11. Cenário **carga solar**: a 80.000 km da superfície do Sol, a carga tem de encher em
    20 s; de volta à cidade (ainda ≥ 90%), a 150 m/s contra um prédio intacto de 24–34 m, ele
    tem de cair de uma vez — sem carga, o mesmo golpe só o furaria
    ([[2026-09-28-pr-026-carga-solar]]). Screenshots em `e2e-out/carga.png` e
    `e2e-out/carga-desabar.png`. O `__launch(maxWidth, speed, minWidth)` só escolhe prédio
    sem desabar e sem dano na altura do voo.
12. Cenário **atravessar a Terra**: do Sol, com a mira da câmera 1,5° ao lado da Terra (a
    assistência leva ao centro), 2,5 s de disparo têm de abrir um buraco de 100 km ou mais
    com a saída nos antípodas (entrada · saída < −0,99). Mirando Metrópolis de cima, nenhum
    buraco novo ([[2026-09-28-pr-027-visao-atravessa-terra]]). Screenshots em
    `e2e-out/atravessa-sol.png` e `e2e-out/buraco-saida.png`. A mira é corrigida pela
    direção da câmera (`__aimCam`), não pelo rumo do herói.

Referência de 28/09/2026: 3 missões em **81 s**, 921 pontos.

## Quando rodar

Antes de publicar PR que toque voo, câmera, colisão, visão de calor ou missões.
Não roda no CI (precisa de Chrome com GPU).

## Se falhar

- **Travou numa missão**: abra a screenshot dela em `e2e-out/`. Se o jogo parece
  certo e o autopiloto é que não sabe resolver a situação nova, melhore o autopiloto
  **e diga isso no PR** — mudar o teste para passar exige justificar.
- **Erro no console**: é bug. O e2e não tolera `pageerror`.
