---
tipo: adr
numero: 3
data: 2026-09-28
status: aceito
tags: [adr, render, espaco, precisao]
---

# ADR-003 — Origem flutuante: o mundo num grupo deslocado

## Contexto

A fase 2 do [[roadmap]] leva o herói ao espaço em escala real: a Terra tem 6.371 km de raio
e o Sol fica a 1,5×10¹¹ m. A GPU trabalha em float32, com 24 bits de mantissa. Numa posição
absoluta de 10¹¹ m, a resolução é de **~18 km**. O three monta `modelViewMatrix` em float64 na
CPU, então a posição na tela sai certa. Mas tudo que o shader calcula em **posição de mundo**
vira lixo: reflexo do mapa de ambiente, coordenada de sombra, luz pontual.

## Decisão

- **Tudo que tem posição no mundo mora num grupo `world`**, e a câmera é filha dele. As
  posições continuam verdadeiras (metros, referencial da cidade, float64 no JS), e o grupo é
  deslocado por −origem. As matrizes de mundo que chegam à GPU são a diferença, calculada em
  float64: números pequenos.
- **Perto da cidade (< 20 km) a origem é zero**: o jogo que já existia não muda. Além disso,
  a origem acompanha o herói a cada quadro.
- **Os módulos não sabem da origem.** Recebem o contêiner no lugar da cena e convertem com
  `localToWorld`/`worldToLocal` onde misturam espaços:
  - `lookAt` quer o alvo em espaço de render (câmera e feixes da visão de calor);
  - `getWorldPosition` devolve espaço de render (os olhos, na visão de calor);
  - a capa calcula os pinos a partir do `matrixWorld` e guarda os vértices **relativos** ao
    mesh, que fica na origem da capa.
- **Neblina e mapa de ambiente são da `Scene` de verdade.** Num grupo, o renderer os ignora.
- A origem é atualizada logo depois do voo e antes de qualquer conversão. Senão, a conversão
  usa a matriz do quadro anterior (a 1.000 km/s, 17 km de erro).

## Consequências

- ✅ A 5×10⁸ m da cidade, a câmera fica a ~5 m da origem de render (verificado no e2e), e o
  herói, a capa e os feixes saem inteiros.
- ✅ Longe da cidade, a cidade some pelo `far` da câmera, e o deslocamento não custa nada a
  ela.
- ⚠️ Código novo que use `lookAt`, `getWorldPosition` ou `matrixWorld` para posicionar algo
  precisa converter entre espaços. Vértice com posição absoluta num `Float32Array` perde
  precisão longe: guarde relativo ao mesh.
- ⚠️ Colisão, raycast e simulação continuam em coordenadas verdadeiras: não mudam.

Implementado no [[2026-09-28-pr-023-origem-flutuante]].
