---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 22
url: https://github.com/VictorNascimento14/Superman/pull/22
branch: feat/desabamento
tags: [pr, destruicao, cidade, colisao]
status: merged
---

# PR #22 — Desabamento

## 🎯 Contexto

Segunda peça da fase 2 do [[roadmap]]. Depois do [[2026-09-28-pr-021-atravessar-predios]],
furar um prédio já deixava rombos, mas ele continuava de pé. Com dano de verdade, a parte de
cima tem de cair, como aconteceria.

## 🔧 Mudanças

- `src/world/damage.js` (novo, puro): cada travessia tira
  `6 m / largura × (1 + v/200)` do andar, somado por faixa de 8 m. Com 50% numa faixa, o
  prédio desaba a partir da base dela.
- `src/world/buildingGeo.js` (novo, sem DOM): as primitivas de montagem saíram do `city.js`
  (`GeoBuilder`, `addWalls`, `addTop`, `addBox`), mais o `cutTier`, que corta um nível na
  malha mesclada sem esticar as janelas.
- `src/world/city.js` — guarda, por prédio, o primeiro vértice de cada nível em cada malha
  (`refs`) e os objetos de telhado dele. Expõe `setTop` (encurta no lugar, só as faixas do
  prédio sobem para a GPU), `makePart` (a parte de cima, idêntica) e `roofProps`.
- `src/world/collapse.js` (novo): a queda, com 7 m/s², tombamento até ~8°, linha de
  esmagamento, colisão acompanhando, objetos de telhado caindo junto, poeira e entulho dos
  lados, e no fim o toco de 3 m com escombros.
- `src/world/layout.js` — cada caixa de prédio sabe de que prédio é (`building`).
- `src/fx/instances.js` (novo) — `hideInstancesIn`, que esconde furos e queimados da parte
  que caiu (`breach.js` e `heatvision.js` ganham `clearMarks`).
- `src/fx/breach.js` — `burst` e `puff` genéricos, e duração, tamanho e opacidade por
  nuvem.
- `src/main.js`, `src/audio/audio.js` — ronco longo, tremor por proximidade e o aviso
  "DESABOU!".
- Testes: 5 de dano, 3 de corte (contra a geometria real) e 1 de marcas. O e2e ganha o
  cenário **desabar**, e a busca de alvo passa a olhar as quatro faces a 12 m da rua.

## 🧠 Decisões técnicas

- **Cortar no lugar, não trocar a malha.** Os prédios são 4 malhas mescladas por estilo.
  Como cada parede é um quadrilátero só, encurtar é mexer em 8 vértices por nível e subir
  só essa faixa (`addUpdateRange`). Nada de reconstruir a cidade.
- **O bloco que cai afunda sob a laje.** A laje de 4 m é opaca, então esconde o bloco sem
  plano de recorte nem shader novo, portanto sem compilar programa no meio do jogo. A poeira
  cobre a linha de esmagamento.
- **Dano por faixa de altura**, e não um total do prédio: é o mesmo andar que precisa ceder.
- **O Planeta Diário não cai.** O globo e o letreiro são objetos à parte e não acompanhariam
  o bloco.

## Verificação

Travessia supersônica num prédio de 68 m e 34 m de largura: "DESABOU!" na hora, teto da
colisão em 22 → 15 → 3 m, desabamento concluído em ~4,5 s, 440 pedaços de entulho e
console limpo.

![[desabamento.jpg]]

## ⚠️ Armadilhas e aprendizados

Registradas em [[destruicao]]:

- caixa de altura zero vira placa invisível no ar (o nível que sumiu vai para debaixo da
  terra);
- marcas coladas em prédio que caiu ficam no ar;
- a parte que cai precisa ser montada com y absoluto para as janelas baterem.

No roteiro de fotos: a 40 m da fachada nenhuma face tem rua livre (0 de 292), porque o ponto
cai no quarteirão vizinho; a 12 m são 146.

## 🧪 Como testar

1. `npm test` (67), `npm run build` e `npm run e2e`: missões com 921 pontos; atravessar com
   3 rupturas e 110 pedaços; e desabar, com o prédio caído, a animação concluída e o teto em
   3,0 m.
2. `npm run dev`: segure Shift até o supersônico e atravesse um prédio estreito. Ele desaba
   atrás de você. Em cruzeiro, fure o mesmo andar duas vezes.

## 📎 Documentação afetada

- [[destruicao]], [[cidade-procedural]], [[colisao]], [[runbook-e2e]]
- [[roadmap]], [[arquitetura]]
- [[2026]] (changelog)
