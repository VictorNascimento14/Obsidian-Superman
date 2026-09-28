---
tipo: aprendizado
data: 2026-09-28
tags: [aprendizado, desempenho, medicao]
---

# FPS no Chrome headless não mede ganho de desempenho

## Sintoma

Medindo quadros por 4 s em três cenas (`?q=alta`), a **mesma versão** deu 60, 33,
31 e 58 fps em rodadas seguidas. Antes/depois de uma otimização, a variação entre
rodadas era maior que a diferença entre versões.

## Causa

O headless desta máquina (780M, Vulkan) sincroniza e estrangula quadros de forma
irregular — aquecimento, outras abas, gerenciamento de energia. Um número de FPS
isolado não diz nada.

## O que fazer

- **Declarar o trabalho removido, não o FPS**: triângulos e draw calls vêm de
  `renderer.info` (com `autoReset = false` — ver [[pipeline-de-render]]) e são
  determinísticos. Foi assim no [[2026-09-28-pr-013-recorte-e-agua]]: −30% de
  triângulos, FPS "inconclusivo".
- Para FPS de verdade: navegador com tela, várias rodadas, mediana.
