---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 18
url: https://github.com/VictorNascimento14/Superman/pull/18
branch: fix/mundo-chao-e-pedestres
tags: [pr, mundo, colisao, trafego]
status: merged
---

# PR #18 — Mundo: chão da colisão igual ao da cena, pedestres no chão e faixas sem ultrapassagem

## 🎯 Contexto

Achados da revisão do [[2026-09-28-pr-014-revisao-de-codigo]] na fatia de mundo. Os três
primeiros têm a mesma causa: a cena e a colisão tinham números próprios para o chão.

## 🔧 Mudanças

- `src/world/layout.js` — `QUAY` (30 m de cais), `WATER_Y` (−1,2) e `LAWN` (0,6) viram a
  fonte única do chão. `collisionBoxes` ganha a laje da ilha (topo em 0) e o gramado do
  parque.
- `src/world/collision.js` — `createCollisionWorld(boxes, { cellSize, floor })`: o piso
  infinito, que era fixo em y = 0, passa a ser configurável (esfera, raio e `heightAt`).
  No jogo ele fica no nível do mar (`main.js`).
- `src/world/city.js` — laje, gramado, lago, árvores e água usam as constantes.
- `src/world/trafficView.js` — o pedestre tem a origem nos pés e o `update` só soma o
  balanço do passo.
- `src/world/traffic.js` — velocidade fixa por faixa (saiu o ±5% por carro) e nenhum carro
  nasce a menos de 6 m de outro da mesma faixa.
- `tests/layout.test.js` — contagem de caixas atualizada, e um teste amarra o chão da
  colisão ao da cena: rua 0, gramado `LAWN`, mar `WATER_Y`, raio parando na água.

## 🧠 Decisões técnicas

- **A laje como caixa, e não um piso "por região".** O teste de slab já cobre o topo e as
  paredes do cais. Um piso por partes teria de reimplementar as duas coisas no raycast.
  O custo é uma caixa a mais por consulta, e o carimbo (`stamp`) evita testá-la duas vezes
  no mesmo raio.
- **O piso vira opção de `createCollisionWorld`, com padrão 0.** Os testes de voo e de
  colisão, que montam mundos pequenos sobre y = 0, continuam valendo sem mudança.
- **A origem do pedestre nos pés, em vez de compensar no `update`.** A
  `CapsuleGeometry(0,22; 0,9)` mede 1,34 m (a altura passada é só o trecho reto). Com o
  centro a 0,78, a base ficava a 0,11 m, e o `+0,15` do update levava os pés a 0,26–0,31 m.
- **Espaçamento no nascimento junto com a velocidade fixa.** Sem ele, os 13 pares que
  nasciam sobrepostos andariam grudados para sempre.

## Medição

Pares de carros na mesma faixa e reta a menos de 4,4 m (um carro), em 60 s simulados:

| | Pares sobrepostos | Já no nascimento |
|---|---|---|
| Antes (±5% por carro) | 141 | 11 |
| Só velocidade fixa | 41 | 13 |
| Velocidade fixa + espaçamento | **30** | 0 |

Os 30 restantes saem juntos da curva na mesma faixa, porque ninguém cede a vez. O teto
ficou anotado no código (`ponytail:`), com o caminho de upgrade: reservar a faixa de saída.

## ⚠️ Armadilhas e aprendizados

- Número mágico repetido entre render e colisão diverge em silêncio: 0,6 e −1,2 viviam só
  no `city.js`.
- O cofre descrevia a velocidade como "±5%" e, na mesma frase, dizia que "ninguém
  ultrapassa". As duas coisas não cabiam juntas.

## 🧪 Como testar

1. `npm test` (46), `npm run build` e `npm run e2e` (anéis 1 · resgate 1 · drones 1, 921 pontos).
2. `npm run dev`: pouse no parque (pés na grama) e sobre o mar (pés na água); desça numa
   calçada e olhe os pedestres de perto.

## 📎 Documentação afetada

- [[colisao]], [[cidade-procedural]], [[trafego-e-pedestres]]
- [[2026]] (changelog)
