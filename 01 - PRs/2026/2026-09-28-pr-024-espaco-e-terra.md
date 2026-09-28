---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 24
url: https://github.com/VictorNascimento14/Superman/pull/24
branch: feat/espaco-terra
tags: [pr, espaco, voo, render, hud]
status: merged
---

# PR #24 — Subir ao espaço e ver a Terra

## 🎯 Contexto

Peça 13 da fase 2 do [[roadmap]]: do chão de Metrópolis ao espaço, em escala real (escolha do
jogador), vendo a Terra. Decisões em [[ADR-004-espaco-em-escala-real]]; o alicerce foi a
[[ADR-003-origem-flutuante]].

## 🔧 Mudanças

- `src/space/nav.js` (novo, puro): `altitude`, `fromCity`, `nearCity`, `upAt`, `speedCap`
  e as constantes `SPACE`.
- `src/player/flight.js`:
  - hipervelocidade acima de 20 km (alvo `1,2 × altitude`);
  - teto duro `2,5 × altitude`;
  - colisão só no mundo plano, piso a 30 km fora dele;
  - `flight.altitude`.
- `src/space/space.js` (novo): a cena do espaço, em espaço escalado. A Terra é procedural
  (continentes, calotas, oceano especular, nuvens, luzes das cidades, atmosfera), e a ilha
  de Metrópolis é desenhada no mesmo grid da cidade. Estrelas.
- `src/render/post.js` — passada do espaço antes da cidade, ligada acima de 2,5 km.
- `src/render/sky.js`:
  - `setSpace(k, low)`: céu e cúpula da noite esmaecem, e o céu de baixo mais cedo;
  - a luz azul do céu e o reflexo dele apagam;
  - `sunTrue` para o globo.
- `src/main.js`:
  - transição por altitude (espaço, céu, cidade, neblina que afina) e som sem vento no
    vácuo;
  - marcador de Metrópolis na tela.
- `src/ui/hud.js` — altitude em km, km/s (e múltiplos da luz), modo **HIPERVELOCIDADE**,
  `setBeacon`.
- Testes: 3 de navegação e 4 de voo no espaço (subida exponencial, descida sem atravessar,
  piso fora da cidade, o outro lado da Terra). O e2e ganha o cenário **espaço** (ida e
  volta).

## 🧠 Decisões técnicas

- **Cena separada em espaço escalado**, e não `logarithmicDepthBuffer`. O buffer
  logarítmico valeria para a cidade inteira e desligaria o early-z. Com o espaço escalado,
  um buffer comum serve de 1 m a 10¹² m.
- **Velocidade proporcional à distância**: exponencial na ida e suave na volta, sem
  mecânica de "frear" para aprender.
- **A ilha desenhada no mesmo grid da cidade.** A troca entre cidade de verdade e globo
  acontece aos 4 km, e as ruas desenhadas caem no mesmo lugar das de verdade.
- **Luzes de cidade em duas escalas**: regiões de longe e pontos de perto. Ponto fino visto
  de longe só serrilha, e região vista de perto parecia nuvem acesa.
- **Shader barato**: hash aritmético sem `sin`, oitavas por camada, detalhe e luzes só onde
  aparecem.

## Verificação

![[espaco-subida.jpg]]

Sequência sobre Metrópolis (dia e noite): a 3 km aparece a cidade de verdade; aos 5 km, a
ilha desenhada; a 400 km, de noite, as luzes; a 7.000 km, a Terra com o marcador. Console
limpo em todas.

## ⚠️ Armadilhas e aprendizados

Registradas em [[espaco]]:

- a luz da cidade não é o Sol (o globo aparecia de dia à noite);
- o chão plano do outro lado da Terra;
- o branco do céu Preetham abaixo do horizonte;
- "atrás da câmera" se testa antes da projeção;
- o TDZ no `main.js`, pego capturando `pageerror`.

Com o pitch em 83°, subir mirando deriva centenas de km. Daí o marcador, e o teste de descida
usa a tecla de descer (C).

## 🧪 Como testar

1. `npm test` (75) e `npm run build`. `npm run e2e`: tudo de antes, mais o cenário **espaço**
   (subiu a 6.866 km em 8 s; desceu a 1 m em 15 s).
2. `npm run dev`: segure Espaço + Shift. A cidade fica para trás, o céu escurece e a Terra
   aparece. O marcador ◇ METRÓPOLIS aponta a volta; segure C + Shift para descer. Aperte T
   para ver a Terra de noite.

## 📎 Documentação afetada

- [[ADR-004-espaco-em-escala-real]] (nova), [[espaco]] (nova)
- [[voo-e-camera]], [[pipeline-de-render]], [[hud-e-minimapa]], [[audio]], [[runbook-e2e]]
- [[roadmap]], [[arquitetura]], [[2026]] (changelog)
