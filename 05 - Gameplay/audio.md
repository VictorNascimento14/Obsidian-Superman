---
tipo: sistema
data: 2026-09-28
area: gameplay
arquivos: [src/audio/audio.js, src/audio/curves.js]
tags: [sistema, audio]
---

# Áudio procedural

Nenhum arquivo de som: tudo é sintetizado com WebAudio (ruído filtrado, osciladores,
envelopes). **M** liga e desliga.

## Camadas contínuas (atualizadas por quadro)

| Som | Como | Controlado por |
|---|---|---|
| Vento | ruído → passa-faixa | velocidade: volume 0,015 → 0,32 e filtro 250 → 2.150 Hz até 160 m/s |
| Cidade | ruído → passa-baixa 380 Hz | altura sobre o solo: some aos 60 m |
| Visão de calor | 3 serras (110, 113,5, 220 Hz) → passa-baixa 900 Hz | disparando |
| Chiado | ruído → passa-alta 4 kHz | disparando **e acertando** algo |

As curvas vivem em `curves.js` (puras, testadas).

## Eventos

| Evento | Som |
|---|---|
| Estrondo sônico | ruído com varredura 1.400 → 60 Hz + "thump" senoidal 70 → 30 Hz |
| Impacto | igual, mais curto, volume pela velocidade |
| Explosão de drone | ruído 1.200 → 100 Hz + thump |
| Decolagem | sopro agudo curto |
| Anel / pegou a pessoa | duas notas (880, 1.320 Hz) |
| Missão cumprida | arpejo Dó-Mi-Sol-Dó em triângulo |
| Missão falhou | Sol-Mi-Dó descendo |
| Alerta (missão nova, "ele caiu!") | bipe alternado em quadrada |

Um **compressor** no fim da cadeia impede que estrondo e explosão juntos estourem.

## Armadilhas

- **`AudioContext` só nasce de gesto do usuário**: é criado no clique de "Clique para
  voar" (`audio.start()`), nunca no carregamento.
- **Verificar som sem ouvir**: `audio.probe()` mede o RMS da saída por um
  `AnalyserNode`. Medido em headless: parado 0,016 → 376 m/s 0,044; visão de calor
  0,007 → 0,062.

## PRs

- [[2026-09-28-pr-012-audio]]
