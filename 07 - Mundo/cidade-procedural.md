---
tipo: sistema
data: 2026-09-28
area: mundo
arquivos: [src/world/layout.js, src/world/city.js, src/world/textures.js]
tags: [sistema, mundo, cidade]
---

# Cidade procedural (Metropolis)

![[cidade-noite.jpg]]

## O que faz

Gera uma Metropolis numa ilha a partir de uma **semente fixa** (1938): mesma
semente, mesma cidade, sempre — coberto por teste.

## Como funciona

**`layout.js` (lógica pura, testada)**

- Grade de **20 × 20 quarteirões** de 90 m com ruas de 22 m → ilha de 2,24 km.
  O eixo de cada rua fica na borda da célula de 112 m (`streetLine(k)`).
- **Densidade** cai com a distância ao centro financeiro (`DOWNTOWN`), e dela saem
  a altura máxima (`16 + 320·t^1.8` m), o loteamento (1 lote no centro, 3×3 no
  subúrbio) e o estilo (`vidro`, `artdeco`, `concreto`, `tijolo`).
- Prédios altos ganham **recuos** (setbacks) em 2 ou 3 níveis, no desenho dos
  arranha-céus dos anos 30.
- **Parque** 3×4 quarteirões: as ruas internas ficam fechadas (`streetOpen`),
  o que o tráfego vai respeitar.
- **Planeta Diário** no quarteirão (9, 9): quatro recuos até 262,8 m, letreiro e
  globo dourado de 14 m de raio sobre um pedestal.
- `collisionBoxes()` devolve uma AABB por nível (+ o globo) para a [[colisao]].

**`city.js` (meshes)**

- Paredes **mescladas por estilo** (4 meshes para ~2.000 prédios), telhados num
  mesh, cornijas noutro. UV em metros de fachada: janela tem o mesmo tamanho em
  qualquer prédio. **Tom por prédio em vertex color** multiplica a textura.
- Chão: uma laje de 4 m (a borda vira o cais) com a textura de rua repetida por
  célula. Água em volta com normal map procedural animado. Margem do cais, nível da
  água e altura do gramado vêm de `layout.js` (`QUAY`, `WATER_Y`, `LAWN`) — os mesmos
  da [[colisao]].
- Instanciados: ~700 árvores, 1.764 postes, caixas d'água, condensadoras, antenas
  com luz de aviso piscando.
- `update(dt, time, night)` acende janelas, postes e letreiro conforme `night` do
  [[pipeline-de-render]].
- Montar e cortar prédio mora em `buildingGeo.js` (sem DOM, testado). A cidade guarda,
  por prédio, onde estão os vértices de cada nível (`refs`) e os objetos de telhado dele,
  e expõe `setTop`, `makePart` e `roofProps` para o [[destruicao|desabamento]], e `setOpen`
  para o recorte dos furos nos prédios com [[interior-dos-predios|interior]] montado.

**`textures.js`** — tudo em canvas: fachada 8×8 vãos (cor + emissivo das janelas
acesas + roughness/metalness numa textura só: G e B), chão com faixas, meio-fio e
faixas de pedestre, grama, telhado, ondas, letreiro.

## Números

| Item | Valor |
|---|---|
| Prédios | 2.064 (35 acima de 150 m) |
| Draw calls da cena | ~45–65 (sombra incluída) |
| Triângulos | ~430 mil |

## Armadilhas

- **Parede de face única e sombra**: por padrão, só a face de trás entra no mapa de sombra, e
  o miolo do prédio ficava ao sol (o interior saía lavado). As paredes fazem sombra dos dois
  lados (`shadowSide`) — [[2026-09-28-pr-028-interior-dos-predios]].

- **Cornija com tampa cobre o telhado.** A caixa da cornija tinha face de cima e
  pintava o telhado inteiro de pedra clara — de cima, a cidade inteira parecia
  branca. Cornija agora é só as quatro faces (parapeito).
- **Tom em vertex color só muda o matiz se a textura for neutra.** Com o vidro já
  azul na textura, todo prédio de vidro saía no mesmo azul.
- **`mergeGeometries` exige todos indexados ou nenhum**: `IcosahedronGeometry` não
  é indexado, `CylinderGeometry` é → `toNonIndexed()` antes.
- **Ladrilho 4×4 repetia de perto**: à noite apareciam colunas de janelas acesas
  iguais a cada 16 m. Com 8×8 e deslocamento de 1/8 por prédio, some.

## Galeria

![[cidade-dia-parque.jpg]]
![[cidade-entardecer.jpg]]

## PRs

- [[2026-09-28-pr-003-cidade-procedural]]
