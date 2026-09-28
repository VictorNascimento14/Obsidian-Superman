---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 27
url: https://github.com/VictorNascimento14/Superman/pull/27
branch: feat/visao-atravessa-terra
tags: [pr, poderes, espaco, visao-de-calor, render]
status: merged
---

# PR #27 — A visão de calor carregada atravessa a Terra

## 🎯 Contexto

Peça 16, a última da fase 2 do [[roadmap]]: "se eu usar a visão de calor na Terra e eu
estiver perto do Sol, o poder é tão grande que atravessa a Terra e deixa o buraco". O jogador
escolheu que desse para atirar do próprio Sol, com mira assistida no marcador, ou voltar
carregado e atirar de perto. Depende da [[carga-solar]] ([[2026-09-28-pr-026-carga-solar]]).

## 🔧 Mudanças

- `src/powers/pierce.js` (novo, puro):
  - `assistAim`: no disco da Terra, mira livre; a até ~3° dele, ao centro;
  - `chord`: entrada e saída do raio pela esfera;
  - `createPierce().update`: abre buracos, alarga o que está sob o raio e o faz seguir o raio;
    Metrópolis protegida a 150 km.
- `src/powers/heatvision.js` — `update(dt, wants, over)`: a direção assistida no lugar da mira
  da câmera.
- `src/space/space.js`:
  - `setBeam`: o raio na cena do espaço, com espessura aparente constante — um cone do herói
    até a entrada e um cilindro da saída para além;
  - `setHoles`: até 8 buracos (entrada e saída) no shader do globo — túnel, borda derretida em
    HDR e chão queimado.
- `src/main.js`:
  - o disparo decidido no próprio quadro, só fora do mundo plano;
  - avisos A TERRA FOI ATRAVESSADA! (com estrondo) e METRÓPOLIS, NÃO.
- `README.md` — a fase 2 na descrição, e os comandos da visão de calor e do som.
- Testes: 6 em `pierce.test.js` (corda do Sol, mira assistida, buraco que cresce e fica,
  mira que treme e o buraco que segue, o nono buraco, Metrópolis e carga mínima). Total 96.
- e2e: cenário **atravessar a Terra**, com a mira corrigida pela câmera (`__aimCam`).

## 🧠 Decisões técnicas

- **Metrópolis protegida.** A cidade de verdade é plana (o recorte sobre a esfera); um buraco
  nela existiria no globo e não na cidade. O raio para na superfície e avisa.
- **Mira assistida pelo disco, não por pixel.** Do Sol, a Terra tem 9 px. Com o ◇ TERRA a até
  ~3° da mira, o raio vai ao centro; perto, onde se mira.
- **O buraco segue o raio.** Com o critério "dentro do raio", o ajuste da câmera de ombro ao
  começar a mirar abria 8 buracos em fila. A mira que anda até `max(2 × raio, 250 km)` continua
  no mesmo buraco, que desliza atrás dela.
- **Corda sem `b² − c`.** Do Sol, os termos têm 22 casas e a diferença (13) sumiria. A
  distância de passagem é medida direto, e o ângulo da mira sai por `atan2(|a × b|, a · b)`.
- **Espessura aparente constante.** O cone com a ponta no herói e o cilindro com o raio
  proporcional à distância dão ~0,0012 rad de perto até 1 UA; com o bloom, o raio brilha
  inteiro.

## Verificação

![[terra-atravessada.jpg]]

Em cima: do Sol, o raio sai rumo ao ◇ TERRA; de 2 raios da Terra, o raio entra no planeta.
Embaixo: a entrada no lado do dia e a saída nos antípodas, no lado da noite, entre as luzes
das cidades.

- e2e: do Sol, com a mira da câmera 1,5° ao lado da Terra, 2,5 s de disparo abriram **1
  buraco de 124 km** com entrada · saída = **−1,000** (antípodas). Mirando Metrópolis de
  1.000 km, nenhum buraco novo. Console limpo.

## ⚠️ Armadilhas e aprendizados

Registradas em [[atravessar-a-terra]]:

- a mira é a câmera, não o rumo do herói (a câmera de ombro olha ~4° abaixo);
- o `firing` do quadro anterior abria buraco depois de teleporte;
- buraco por quadro virava fila;
- na cidade o raio carregado para no chão (senão o aviso de Metrópolis saía a cada disparo);
- a mira gira ~1° ao começar a disparar (câmera de ombro): de 14.000 km, o ponto no chão anda
  300 km. O e2e mira Metrópolis de 1.000 km.

## 🧪 Como testar

1. `npm test` (96) e `npm run build`. `npm run e2e`: tudo de antes, mais **atravessar a
   Terra**.
2. `npm run dev`:
   - carregue no Sol (peça 15);
   - vire até o ◇ TERRA ficar perto da mira e segure F: o raio cruza o espaço e aparece
     **A TERRA FOI ATRAVESSADA!**;
   - voe até a Terra (◇ TERRA): a cratera em brasa está no lado que olhava para o Sol, e a
     saída, do outro lado, brilha de noite.

## 📎 Documentação afetada

- [[atravessar-a-terra]] (nova)
- [[visao-de-calor]], [[carga-solar]], [[espaco]], [[runbook-e2e]]
- [[roadmap]] (fase 2 concluída), [[arquitetura]], [[2026]] (changelog)
