---
tipo: aprendizado
data: 2026-09-28
tags: [aprendizado, desempenho, gpu, medicao]
---

# Tirar da cena não libera a GPU

## Sintoma

Nada visível: o jogo rodava igual. Mas cada missão de drones deixava ~50 geometrias
(e ~30 materiais) presas na GPU para sempre. Medindo `renderer.info.memory.geometries`
no início de cada missão, pulando 9 seguidas (3 ciclos):

| Ciclo | Antes — anéis · resgate · drones | Depois |
|---|---|---|
| 1 | 75 · 75 · 126 | 75 · 75 · 126 |
| 2 | 126 · 126 · 176 | 76 · 76 · 126 |
| 3 | 176 · 176 · 226 | 76 · 76 · 126 |

## Causa

`root.remove(obj)` só tira o objeto do grafo de cena. Os buffers da geometria e o
programa do material continuam alocados no WebGL até alguém chamar
`geometry.dispose()` e `material.dispose()` — e `material.dispose()` **não** libera as
texturas do material, que pedem `texture.dispose()` à parte. Cada missão cria peças
próprias (5 drones × 10 peças, 8 anéis com material próprio, a pessoa do resgate),
então o vazamento crescia com o tempo de jogo.

## O que fazer

- **Quem cria objeto temporário descarta.** Em `missions.js`, `end()` chama
  `disposeTree(o)` em tudo que sai da cena.
- **Recurso compartilhado fica de fora.** O `ringGeo` dos anéis é marcado com
  `userData.shared = true` e poupado. Descartar algo ainda em uso não quebra — o three
  sobe de novo no quadro seguinte —, mas troca economia por engasgo (reupload,
  recompilação de shader).
- **Medir o patamar, não o fim.** `window.__game.debugInfo().geometries` expõe o
  contador (não depende do `autoReset` do [[pipeline-de-render]]). Geometria só conta
  depois de desenhada uma vez: o feixe e o pool de explosões entram no primeiro uso, por
  isso "fim ≤ começo" reprova à toa. O certo é o número voltar ao mesmo patamar a cada
  ciclo.

Roteiro usado (puppeteer-core do projeto contra o `vite preview` do build):

```js
await page.evaluate(() => window.__game.start());
for (let k = 0; k < 9; k++) {
  await page.waitForFunction(() => window.__game.missions.current, { polling: 100 });
  await new Promise((r) => setTimeout(r, 700)); // só conta depois de subir para a GPU
  out.push(await page.evaluate(() =>
    `${window.__game.missions.current.type}:${window.__game.debugInfo().geometries}`));
  await page.evaluate(() => window.__game.missions.skip());
}
```

Para o "antes", uma `git worktree` do `main` em pasta temporária — sem `git stash` na
árvore de trabalho. Visto no [[2026-09-28-pr-014-revisao-de-codigo]].
