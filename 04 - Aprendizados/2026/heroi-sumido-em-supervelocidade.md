---
tipo: aprendizado
data: 2026-09-28
tags: [aprendizado, camera, depuracao]
---

# Herói "sumido" em supervelocidade

## Sintoma

Na screenshot a 400 m/s só aparecia céu e mar: nada do herói.

## Causa

Não era bug de render. Projetar a posição do herói (`pos.project(camera)`) deu
NDC ≈ (0; −0,07), bem no centro e a 15 m. O herói estava lá, mas:

1. **deitado na direção do voo, visto exatamente de trás** — pés para a câmera, uma
   silhueta de poucos pixels;
2. **FOV aberto (84°) e 15 m de distância** somados encolhiam o que sobrava.

## Correção

Câmera mais alta com a velocidade (+3,2 m no teto), distância máxima de 15 → 9 m e
FOV máximo de 84° → 76°.

## Como evitar

Antes de caçar bug de render, **projete o objeto na tela**: se o NDC está dentro de
[−1, 1] e a profundidade é a esperada, o problema é enquadramento, não desenho.
A sonda é uma linha no gancho `window.__game`.
