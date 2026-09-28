---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 14
url: https://github.com/VictorNascimento14/Superman/pull/14
branch: fix/revisao-de-codigo
tags: [pr, revisao, desempenho, missoes]
status: merged
---

# PR #14 — Revisão de código: GPU liberada ao fim das missões e limpeza

## 🎯 Contexto

Primeira revisão do projeto inteiro, feita depois do [[2026-09-28-pr-013-recorte-e-agua]].
Juntou o `ocr` em modo delegado sobre o range desde o bootstrap, o ESLint via `npx` (sem
entrar no projeto) e uma releitura completa em três fatias: player e poderes; mundo e
render; missões, HUD, áudio e main. Este PR
fecha o vazamento de GPU e os achados de higiene. Os bugs de sistema achados na mesma
revisão (missões, pausa, voo, câmera) vão em PRs próprios.

## 🔧 Mudanças

- `src/game/missions.js` — `end()` descarta a geometria e o material de tudo que a
  missão criou (`disposeTree`). O `ringGeo` compartilhado é marcado com
  `userData.shared` e fica de fora.
- `src/main.js` — `debugInfo()` expõe `geometries` (`renderer.info.memory.geometries`)
  para medir vazamento.
- `src/world/trafficView.js` — o recorte copiava a cor com `subarray`, o que criava uma
  view nova por carro ou pedestre visível a cada quadro (até ~2.300). Agora usa
  `setColorAt(n, col.fromArray(...))`, sem alocar. O buffer de cor fica garantido mesmo
  com 0 instâncias: o módulo já aceitava 0 via `Math.max(1, n)`, mas quebrava ao copiar
  a cor.
- `src/powers/heatvision.js` — as fagulhas no alvo não fazem mais `dir.clone()` por
  quadro.
- `src/player/hero.js`, `src/player/flight.js` — `FOOT_DEPTH` (exportado e nunca
  importado) saiu. O valor fica só em `FLIGHT.footDepth`.
- Código morto: `tmpE` e `tmpQ` (hero), `gray` (textures).

## 🧠 Decisões técnicas

- **`footDepth` mora na física, não no modelo.** A primeira versão da correção fez
  `flight.js` importar de `hero.js`. Os 42 testes passavam, mas a física (lógica pura,
  ADR-001) passava a depender de um módulo de render: uma `CanvasTexture` no topo do
  `hero.js` bastaria para quebrar o `node --test`. A cópia morta era a do `hero.js`, e foi
  ela que saiu.
- **`setColorAt` com `Color.fromArray`** em vez de copiar à mão: é a API do próprio
  three, e reaproveita o `col` que o módulo já tinha.

## Medição

Geometrias vivas na GPU no início de cada missão, pulando 9 seguidas (3 ciclos):

| Ciclo | Antes — anéis · resgate · drones | Depois |
|---|---|---|
| 1 | 75 · 75 · 126 | 75 · 75 · 126 |
| 2 | 126 · 126 · 176 | 76 · 76 · 126 |
| 3 | 176 · 176 · 226 | 76 · 76 · 126 |

Cada missão de drones deixava ~50 geometrias (5 drones × 10 peças) e ~30 materiais
presos para sempre.

## ⚠️ Armadilhas e aprendizados

- [[remover-da-cena-nao-libera-gpu]] — `root.remove` não libera a GPU. E "fim ≤ começo"
  não serve de teste, porque o feixe e as explosões só sobem no primeiro uso.
- Teste verde não prova camada certa: o import cruzado `flight.js → hero.js` passou nos
  42 testes.

## 🧪 Como testar

1. `npm test` (42), `npm run build` e `npm run e2e` (3 missões).
2. `npm run dev` e, no console, `__game.start()`. A cada missão, rodar
   `__game.missions.skip()`: `__game.debugInfo().geometries` volta ao mesmo patamar a
   cada ciclo (roteiro completo em [[remover-da-cena-nao-libera-gpu]]).

## 📎 Documentação afetada

- [[missoes]], [[trafego-e-pedestres]], [[heroi-e-capa]], [[voo-e-camera]]
- [[remover-da-cena-nao-libera-gpu]]
- [[2026]] (changelog)
