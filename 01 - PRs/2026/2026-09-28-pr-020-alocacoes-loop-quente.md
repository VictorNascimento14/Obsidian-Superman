---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 20
url: https://github.com/VictorNascimento14/Superman/pull/20
branch: perf/alocacoes-loop-quente
tags: [pr, desempenho, gc, capa, colisao]
status: merged
---

# PR #20 — Desempenho: 74% menos lixo por quadro

## 🎯 Contexto

A revisão do [[2026-09-28-pr-014-revisao-de-codigo]] listou uma dúzia de violações da
invariante "nada aloca no loop quente". Antes de corrigir uma a uma, medimos. O jogo
alocava ~317 KB por quadro, e a lista inteira explicava uns 5% disso. O ranking real veio
da amostragem de heap: ver [[perfilar-alocacao-antes-de-cortar]].

## 🔧 Mudanças

- `src/player/cloth.js`:
  - o laço das restrições passa a ser indexado, sem desestruturar no `for…of`, e a raiz
    é feita à mão, sem `Math.hypot`;
  - `gridNormals(pos, cols, rows, out)` faz a mesma conta do `computeVertexNormals` de
    uma `PlaneGeometry` com a mesma grade, sem alocar.
- `src/player/hero.js` — as posições da capa vão direto no `Float32Array` do atributo, e
  as normais saem de `gridNormals`.
- `src/world/collision.js`:
  - os visitantes do `forEachNear` (`pushOut`, `topAt`) são criados uma vez, com o estado
    da consulta no mundo;
  - os eixos do slab viram uma constante de módulo e a troca usa variável temporária;
  - as listas de célula são percorridas por índice.
- `tests/cloth.test.js` — as normais da grade batem com as do three (cosseno > 0,9999
  em todos os vértices, com a capa dobrada pelo vento de voo).

## 🧠 Decisões técnicas

- **Cortar pelo ranking medido, não pela lista.** A capa sozinha era 56% (desestruturação)
  e o `Math.hypot` dela mais 12%. Duas linhas, 69% do total.
- **`gridNormals` refaz a conta do three, não uma aproximação.** A primeira versão usava
  diferenças centrais na grade, mais simples, mas divergia até 65° nas dobras da capa em
  voo (cosseno 0,43). A soma de faces, com a mesma triangulação da `PlaneGeometry`, dá
  cosseno 1,000000, ou seja, sombreamento idêntico.
- **Parar quando o topo vira three.** O que sobrou é quase todo interno
  (`setValueM4`, `getParameters`, `setValueV3f`...).

## Medição

Amostragem de heap (CDP, lixo coletado incluído), voo de cruzeiro, KB/s:

| Onde | Antes | Depois |
|---|---|---|
| `cloth.js` `step` | 11.346 | — |
| `Math.hypot` nativo | 2.501 | — |
| `computeVertexNormals` + `normalizeNormals` | 978 | — |
| `collision.js` (`slab`, `raycast`, closure) | 717 | 190 |
| `hero.js` `update` | 191 | — |
| **Total** | **20.155** | **4.479** |

No build, somando os aumentos de `usedJSHeapSize` por quadro:

| Cenário | Antes | Depois |
|---|---|---|
| Voo de cruzeiro | 317,6 KB/quadro | 81 KB/quadro (−74%) |
| Parado com visão de calor | 314,6 KB/quadro | 76 KB/quadro (−76%) |

## ⚠️ Armadilhas e aprendizados

- [[perfilar-alocacao-antes-de-cortar]]: `for…of` com desestruturação, `Math.hypot`,
  `setXYZ` e `computeVertexNormals` alocam sem parecer.
- **Ficou de fora de propósito**, porque cada item custa ≤ ~50 KB/s, menos de 1,5% do que
  restou: `applyPoses` (52), `drawMap` do HUD (50), `startTurn` do tráfego (42), `update`
  da câmera (35), e os `any`/`consumeLook` do input, os `forEach` da visão de calor, os
  marcadores das missões, o argumento do `hero.update` e as curvas do áudio, que ficam
  abaixo do corte do ranking. Voltam à mesa se a amostragem mostrar outra coisa.
- **Pista aberta**: `getParameters` do three roda a ~0,56 MB/s, ou seja, algum material
  reavalia o programa todo quadro. Não é compartilhamento de material entre tipos de
  objeto (conferido na cena), então a suspeita recai sobre os materiais do
  `postprocessing`.

## 🧪 Como testar

1. `npm test` (48), `npm run build` e `npm run e2e` (anéis 1 · resgate 1 · drones 1, 921 pontos).
2. `npm run dev`: a capa se comporta e sombreia como antes, inclusive em supervelocidade.

## 📎 Documentação afetada

- [[heroi-e-capa]], [[colisao]]
- [[perfilar-alocacao-antes-de-cortar]]
- [[2026]] (changelog)
