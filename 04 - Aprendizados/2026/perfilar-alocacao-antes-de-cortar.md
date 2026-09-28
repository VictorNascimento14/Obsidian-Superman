---
tipo: aprendizado
data: 2026-09-28
tags: [aprendizado, desempenho, medicao, gc]
---

# Perfilar a alocação antes de cortar

## Sintoma

A revisão do [[2026-09-28-pr-014-revisao-de-codigo]] listou uma dúzia de alocações no loop
quente (closures, arrays de eixo na colisão, objetos literais) e a primeira ideia foi
corrigir uma a uma. Medido, o jogo alocava **~317 KB por quadro** (~19 MB/s), e a lista
inteira explicava uns 5% disso.

## Causa

O maior alocador não estava na lista: o passo da capa (`cloth.js`) respondia sozinho por
56%, e o `Math.hypot` dele por mais 12%. Nenhum dos dois *parece* alocar:

- **`for (const [a, b, len, stiff] of cons)`**: a desestruturação passa pelo protocolo de
  iterador e aloca a cada restrição (~300 restrições × 5 iterações × até 4 subpassos por
  quadro);
- **`Math.hypot(dx, dy, dz)`**: o V8 guarda os argumentos num array a cada chamada. Use
  `Math.sqrt(dx*dx + dy*dy + dz*dz)`;
- **`attr.setXYZ(i, x, y, z)`**: os doubles do argumento são encaixotados quando a
  função não é inlinada. Escreva direto no `attr.array`;
- **`geometry.computeVertexNormals()`** a cada quadro: ~14 KB por chamada numa grade de
  7×12.

## O que fazer

- **Medir o total**: `--enable-precise-memory-info` no Chrome e, na página, somar os
  aumentos de `performance.memory.usedJSHeapSize` a cada quadro (os tombos são GC).
- **Achar quem aloca**: amostragem de heap pelo CDP **incluindo o lixo já coletado**. Sem
  isso, a amostragem só mostra objetos vivos, e o lixo de vida curta, que é justamente o
  que interessa, some:

  ```js
  await cdp.send('HeapProfiler.startSampling', {
    samplingInterval: 256,
    includeObjectsCollectedByMinorGC: true,
    includeObjectsCollectedByMajorGC: true,
  });
  // ... alguns segundos de jogo ...
  const { profile } = await cdp.send('HeapProfiler.stopSampling');
  // some selfSize por callFrame.url:lineNumber e ordene
  ```

  Rode contra o `vite` de dev: o código sem minificar dá arquivo e linha.
- **Cortar pelo ranking** e medir de novo depois de cada corte. Quando o topo vira código
  do three, pare.

Resultado no [[2026-09-28-pr-020-alocacoes-loop-quente]]: 20,2 → 4,5 MB/s na amostragem;
317 → 81 KB por quadro no build.
