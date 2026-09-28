---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 17
url: https://github.com/VictorNascimento14/Superman/pull/17
branch: fix/voo-colisao-e-camera
tags: [pr, voo, camera, colisao]
status: merged
---

# PR #17 — Voo e câmera: corte por superfície e câmera fora das paredes

## 🎯 Contexto

Dois achados *high* da revisão do [[2026-09-28-pr-014-revisao-de-codigo]], ambos na
resposta à colisão:

- bater num prédio voando rente à rua lançava o herói parede acima;
- encostado numa parede à direita, a câmera entrava no prédio.

## 🔧 Mudanças

- `src/world/collision.js` — `resolveSphere(p, r, outNormal, onContact?)`: além da soma,
  entrega cada normal unitária por callback. Quem não passa o callback não muda nada.
- `src/player/flight.js` — `clip` corta a velocidade contra cada normal, uma de cada vez.
  É criado uma vez por voo, então não aloca por subpasso. O impacto usa a maior componente
  individual.
- `src/player/camera.js` — o raycast de "parede entre herói e câmera" sai do herói
  (`pivot`), não do ponto deslocado pelo ombro.
- `tests/flight.test.js` e `tests/camera.test.js` — um teste para cada caso, e os dois
  falham no código antigo.

## 🧠 Decisões técnicas

- **Normal por contato em vez de soma.** Chão (0,1,0) mais parede (1,0,0), somados e
  normalizados, dão (0,71; 0,71; 0). O corte `v −= n·(v·n)` sobre essa diagonal converte
  metade da velocidade horizontal em subida. Cortar contra cada plano zera as duas
  componentes, que é o certo para caixas alinhadas aos eixos.
- **Callback opcional em vez de lista de contatos.** Não aloca, não muda a assinatura para
  os outros chamadores, e o `outNormal` somado continua valendo para os testes existentes.
- **Raio da câmera a partir do herói.** A esfera de colisão garante que o centro do herói
  nunca está dentro de prédio. Já o ombro (1,1 m, ou 1,7 m mirando) pode estar, e o teste
  de slab ignora a caixa que contém a origem. De quebra, a visão de calor, que traça da
  câmera, deixa de atravessar a parede nesse caso.

## Medição

Herói rente ao chão contra o prédio de teste, 7 inclinações × 5 posições, com
propulsão contínua por 1,5 s:

| Marcha | Antes: batidas que subiram > 3 m | Depois |
|---|---|---|
| Cruzeiro (48 m/s) | 30 de 35 (pior 9,0 m) | 0 de 35 (pior 1,0 m) |
| Boost (150 m/s) | 5 de 35 (pior 22,4 m) | 0 de 35 (pior 1,0 m) |

Câmera com o herói tangente à parede: antes em (−0,20; 21,9; −6,5), **dentro** do prédio;
depois fora, nos dois modos (com e sem mira).

## ⚠️ Armadilhas e aprendizados

- Soma de normais serve para "empurrar para fora", não para cortar velocidade. Ficou
  registrado em [[colisao]].
- A origem do raio precisa estar fora das caixas: o slab ignora a caixa em que o raio
  nasce.

## 🧪 Como testar

1. `npm test` (45) e `npm run build`. `npm run e2e`: anéis 1 · resgate 1 · drones 1, 921 pontos.
2. `npm run dev`: desça até a rua e acelere contra um prédio com o nariz um pouco para
   baixo. O herói para na parede em vez de escalá-la.
3. Encoste o herói numa parede com ela à direita e segure o botão direito: o prédio não
   some.

## 📎 Documentação afetada

- [[voo-e-camera]], [[colisao]]
- [[2026]] (changelog)
