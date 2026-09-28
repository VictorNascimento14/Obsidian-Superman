---
tipo: adr
numero: 1
data: 2026-09-28
status: aceito
tags: [adr, stack]
---

# ADR-001 — Three.js + Vite, tudo procedural

## Contexto

O jogo precisa rodar no navegador, com cidade grande e um herói animado. A
referência (Spiderbench) usa Three.js, postprocessing, modelos feitos por scripts
Blender e texturas geradas por IA. Nesta máquina não há Blender, e o projeto não
deve depender de asset binário de origem incerta.

## Decisão

- **Three.js** (WebGL2) para render, **postprocessing** para bloom/SMAA/vinheta,
  **Vite** para dev server e build. JavaScript puro com módulos ES, sem framework de UI.
- **Modelos, texturas e cidade gerados em código**: o herói é montado com
  primitivas (cápsulas, esferas) e esqueleto de `Object3D`; texturas de fachada
  saem de `CanvasTexture`; a cidade sai de um gerador com semente fixa.
- **Lógica pura separada de render** (`layout`, física de voo, colisão) para ser
  testável com `node --test`, sem navegador.

## Consequências

- ✅ Repositório leve, reprodutível, sem LFS.
- ✅ Testes rápidos das partes que mais quebram (física e colisão).
- ⚠️ O visual do personagem é estilizado, não fotorrealista. Caminho de upgrade:
  trocar `hero.js` por um `.glb` carregado com `GLTFLoader`, mantendo o esqueleto
  lógico (`rig`) como contrato.
