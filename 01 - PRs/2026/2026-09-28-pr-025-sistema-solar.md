---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 25
url: https://github.com/VictorNascimento14/Superman/pull/25
branch: feat/sistema-solar
tags: [pr, espaco, voo, render, hud]
status: merged
---

# PR #25 — Voar até o Sol e aos planetas

## 🎯 Contexto

Peça 14 da fase 2 do [[roadmap]]: sair da órbita da Terra e voar até o Sol e aos planetas,
todos em escala real (escolha do jogador), com marcadores para achá-los. É o terreno das peças
15 (carga solar) e 16 (visão de calor atravessando a Terra). Decisões em
[[ADR-004-espaco-em-escala-real]], revisada aqui.

## 🔧 Mudanças

- `src/space/bodies.js` (novo, puro):
  - `solarSystem(sunDir)`: Terra, Sol a 1 UA na direção do sol do céu, a Lua e os 7 planetas
    na eclíptica, com os eixos inclinados;
  - `sunlight(bodies, p)`: quanto do Sol chega em p, com a penumbra do eclipse.
- `src/space/nav.js` — `nearestSurface` (o corpo com a superfície mais perto) e `EARTH_ONLY`.
- `src/player/flight.js` — `flight.bodies` e `flight.nearest`:
  - a hipervelocidade, o teto e o piso usam o corpo mais perto;
  - piso de cada corpo: 30 km na Terra, `max(20 km, 1% do raio)` nos outros.
- `src/space/space.js`:
  - espaço escalado em log;
  - o Sol (granulação, manchas, limbo, brilho pelo tamanho na tela) e a coroa;
  - um shader de planeta com 4 tipos;
  - os anéis de Saturno com a sombra do planeta;
  - raio aparente mínimo de 0,9 px;
  - `setBodies`.
- `src/render/sky.js` — `setSunlight`: no espaço, a luz do herói é o Sol visto de onde ele
  está, e some na sombra de um corpo.
- `src/main.js`:
  - o sistema solar é recalculado a cada troca de hora;
  - marcadores de todos os corpos, sem sobrepor;
  - T não muda a hora longe da Terra;
  - poeira na tela e som da cidade só no mundo plano.
- `src/ui/hud.js`:
  - `distance` ("149,6 milhões de km", "4,5 bilhões de km");
  - o corpo mais perto no lugar da altitude;
  - `goalDistance` nas missões.
- Testes: 6 novos em `bodies.test.js` (posições, eclíptica, corpo mais perto, eclipse,
  espaço escalado, formato das distâncias) e 2 de voo (Terra → Sol em 10–60 s sem atravessar;
  mergulho em Júpiter para no piso). Total 83. O e2e ganhou o cenário **sistema solar**.

## 🧠 Decisões técnicas

- **O corpo mais perto comanda, medido até a superfície.** Com a mesma lei da Terra
  (alvo `1,2 × d`, teto `2,5 × d`), qualquer destino vira uma subida exponencial seguida de
  uma chegada suave, sem mecânica de frear. Medido do centro, o Sol (7×10⁸ m de raio)
  "puxaria" o comando cedo demais.
- **Sistema estático, preso à hora do dia.** Numa sessão os planetas quase não andariam. Os
  ângulos das órbitas foram escolhidos para espalhar os corpos pelo céu da Terra. A hora gira
  tudo junto (é a Terra girando); longe dela, T não muda nada, para não tirar o planeta de
  perto do herói.
- **Espaço escalado em log, não tudo a 100 km.** Com vários corpos, trazer todos para a mesma
  distância deixava a profundidade entre eles no acaso. `100 km × (1 + ln(d/100 km)/2)`
  preserva a ordem e cabe tudo de 1 m a 10¹⁶ m antes das estrelas.
- **Brilho do Sol pelo tamanho na tela.** Uma estrela de 9 px precisa de HDR alto para o bloom
  fazê-la brilhar; o Sol cobrindo a tela, com o mesmo brilho, estoura tudo em branco. A
  interpolação é geométrica (14 → 1,8), como um olho que se ajusta.
- **A coroa é calculada por pixel** (a menor distância do raio de visão ao centro), numa casca
  de faces de trás. Um sprite só funcionaria visto de longe; a casca funciona de longe, de
  perto e de dentro.
- **Um shader, quatro tipos por `define`.** Cada tipo compila só o ruído que usa: de perto, o
  planeta cobre a tela, e o custo é por pixel.
- **O eixo dos planetas parte do "para cima" da cidade, não da eclíptica.** A câmera usa sempre
  o horizonte da cidade; com o eixo pela eclíptica (inclinada até 48° com o Sol de "dia"), as
  faixas de Júpiter apareciam em pé. Ninguém mede o eixo de Júpiter no jogo, e todo mundo vê
  as faixas.

## Verificação

![[sistema-solar.jpg]]

Da esquerda para a direita, em cima: o Sol a 1,4 milhão de km, a superfície dele a 20.000 km,
a Lua. Embaixo: Júpiter, Saturno com os anéis abertos e Netuno.

- e2e: de 20.000 km, mirando o Sol com boost, chegou a 96.042 km da superfície em **14,7 s**;
  Saturno desenhado com a sombra do planeta nos anéis. Console limpo.
- Tempo de quadro (headless, 1280 × 720): 16,6 ms em todas as cenas (cidade, órbita, rente ao
  Sol, Júpiter, Saturno), preso no vsync; no espaço, 64–70 draw calls contra 124 na cidade.

## ⚠️ Armadilhas e aprendizados

Registradas em [[espaco]]:

- **O chão da cidade só existe no mundo plano.** Em Júpiter, abaixo do plano da cidade, o
  teste de "câmera dentro de prédio" ligava a poeira na tela inteira (uma névoa marrom que
  parecia bloom) e o som da cidade tocava no máximo. O bug já existia do outro lado da Terra
  desde o [[2026-09-28-pr-024-espaco-e-terra]]. Achado desligando objetos um a um até a névoa
  sobreviver sem a cena do espaço.
- **A coroa vista de dentro passava do `far`** e virava um polígono preto no céu.
- **Planeta acima de 1 no HDR vira névoa**: o bloom o espalha pela tela.

## 🧪 Como testar

1. `npm test` (83) e `npm run build`. `npm run e2e`: tudo de antes, mais o cenário
   **sistema solar**.
2. `npm run dev`: suba com Espaço + Shift. Acima de 20 km aparecem os marcadores. Vire até o
   ◇ SOL, segure W + Shift: em ~15 s você está rente à superfície dele. De lá, os marcadores
   levam a qualquer planeta; ◇ TERRA leva de volta, e perto dela vira ◇ METRÓPOLIS.

## 📎 Documentação afetada

- [[espaco]] (sistema solar, parâmetros, tempos de viagem, armadilhas)
- [[ADR-004-espaco-em-escala-real]] (espaço escalado em log; o corpo mais perto)
- [[voo-e-camera]], [[hud-e-minimapa]], [[pipeline-de-render]], [[audio]], [[runbook-e2e]]
- [[roadmap]], [[arquitetura]], [[2026]] (changelog)
