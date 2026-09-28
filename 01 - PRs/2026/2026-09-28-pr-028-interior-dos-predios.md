---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 28
url: https://github.com/VictorNascimento14/Superman/pull/28
branch: feat/interiores
tags: [pr, destruicao, mundo, render]
status: merged
---

# PR #28 — Prédios cheios por dentro

## 🎯 Contexto

Peça 17, a primeira da fase 3 do [[roadmap]]. O pedido: "quando eu estiver atravessando o
prédio não deve ser algo vazio dentro, quero algo bastante realista". Os prédios eram cascas
de face única com as janelas pintadas na textura, e o furo era um decalque escuro por cima da
parede inteira.

## 🔧 Mudanças

- `src/world/interior.js` (novo, puro): `interiorBand` monta, por andar e com semente própria,
  lajes, pilares, núcleo, divisórias (salas ou planta livre), mesas, armários, luminárias e a
  fachada por dentro. `piecesInSphere` diz o que o herói toca.
- `src/world/interiors.js` (novo):
  - um `InstancedMesh` por tipo;
  - interior nos 3 últimos prédios furados, ±3 andares em volta da entrada;
  - `smash` quebra o que o herói toca e sobe só aquela instância;
  - luz de dentro: um quarto da luz do hemisfério e janelas procedurais.
- `src/world/openings.js` (novo): até 24 furos em uniforms compartilhados; `patch` ensina um
  material a recortá-los (borda recortada no shader).
- `src/world/buildingGeo.js` — atributo por vértice `open` (desligado).
- `src/world/city.js`:
  - `setOpen(bi, on)`;
  - as paredes recortam os furos dos prédios com interior;
  - as paredes fazem sombra dos dois lados.
- `src/fx/breach.js`:
  - o decalque do furo também recorta;
  - `spawn` devolve onde o furo ficou;
  - `burst` aceita a cor do que quebrou.
- `src/world/collapse.js` — o evento de desabamento traz o prédio.
- `src/player/flight.js` — `flight.inside`.
- `src/main.js`:
  - furo de entrada monta o interior e abre os furos;
  - lá dentro, a esfera de 2,2 m quebra o que toca (entulho na cor do material, poeira);
  - desabamento tira o interior;
  - poeira na tela a 12% com interior.
- Testes: 6 em `interior.test.js`. Total 102. O e2e **atravessar** passa a exigir o interior.

## 🧠 Decisões técnicas

- **Interior sob demanda, não em toda a cidade.** São 2.064 prédios; um andar grande tem
  centenas de peças. Montar só no prédio furado, e só em ±3 andares, custa 6,8 ms no pior
  prédio (82 × 82 m), uma vez, no impacto.
- **Semente por andar.** O mesmo pedaço tem o mesmo id em qualquer faixa. Se o herói volta ao
  prédio, ou a faixa acompanha um mergulho, o que ele quebrou continua quebrado.
- **Furo recortado no shader, não na malha.** As paredes de 2.064 prédios estão mescladas em
  4 malhas; recortar geometria exigiria refazer a malha. O descarte testa até 24 furos, e só
  nos prédios com interior (atributo por vértice): o resto da cidade não paga nada.
- **O decalque recorta junto.** O miolo escuro dele some com a parede, e a borda de concreto
  quebrado fica. O raio da abertura é o do miolo desenhado na textura (30% do decalque).

## Verificação

![[interior-dos-predios.jpg]]

Em cima: atravessando de dia (o corredor, as mesas ao fundo) e à noite (as luminárias acesas).
Embaixo: o furo aberto visto de fora, com o andar lá dentro, e o que o herói quebrou voando.

- e2e: a 150 m/s, 3 rupturas e **353 pedaços de entulho** (eram 110). Interior montado, **54
  peças quebradas** lá dentro, 2 furos abertos.
- Travessia a 45 m/s: 16,7 ms por quadro (vsync), um pico de 33 ms no impacto; 120 draw calls,
  466 mil triângulos.
- Cidade vista de cima, de dia e ao entardecer, com e sem a sombra dos dois lados: iguais.

## ⚠️ Armadilhas e aprendizados

Registradas em [[interior-dos-predios]]:

- parede de face única não fazia sombra no próprio prédio, e o interior ficava ao sol;
- a luz do hemisfério não tem sombra e tingia o carpete de azul;
- janela de dentro acima de 1 no HDR virava véu de bloom;
- nenhum prédio da cidade tem menos de 16 m de lado.

## 🧪 Como testar

1. `npm test` (102) e `npm run build`. `npm run e2e`: tudo de antes, com o interior no
   cenário **atravessar**.
2. `npm run dev`: voe de frente contra um prédio (o cruzeiro, a 173 km/h, já fura). Lá dentro passam
   andares, pilares, salas e mesas, e o que você toca voa em pedaços. Depois, pare na frente do
   furo: dá para ver o andar lá dentro. De noite (T), as luminárias estão acesas.

## 📎 Documentação afetada

- [[interior-dos-predios]] (nova)
- [[destruicao]], [[cidade-procedural]], [[runbook-e2e]]
- [[roadmap]] (fase 3), [[arquitetura]], [[2026]] (changelog)
