---
tipo: sistema
data: 2026-09-28
area: gameplay
arquivos: [src/ui/hud.js, src/ui/overlay.js]
tags: [sistema, hud, ui]
---

# HUD e minimapa

![[hud.jpg]]

## O que mostra

| Onde | O quê |
|---|---|
| Canto inferior esquerdo | velocidade (km/h), barra até 420 m/s, altitude sobre o solo e absoluta, estado (EM PÉ, PAIRANDO, VOO, VELOCIDADE, SUPERSÔNICO) |
| Canto inferior direito | minimapa circular |
| Topo | objetivo da missão (some quando vazio) |
| Centro | mira; avisos grandes (`toast`) como "BARREIRA DO SOM" |
| Base, centro | energia da visão de calor |
| H | painel de controles |

## API (`createHud(layout)`)

`show/hide`, `toggleHelp`, `setObjective(texto)`, `setMarkers([{x, z, color}])`,
`setEnergy(0..1)`, `toast(texto, s)`, `update(dt, flight, alturaDoSolo)`.

## Como funciona

- **DOM só é escrito quando o valor muda** (`set` compara com o último): escrever
  texto todo quadro força layout.
- **Mapa-base desenhado uma vez** num canvas 512 px (água, ilha, parque, prédios em
  tons por altura, marco em ouro). Por quadro: recorte circular, rotação e zoom.
- **Frente para cima**: `θ = −π/2 − atan2(cos yaw, sin yaw)` leva a direção do olhar
  para o topo do círculo.
- **Zoom pela velocidade** (3,2× parado → 1× a 300 m/s).
- **Marcadores fora do círculo** ficam presos na borda, na direção certa.

## PRs

- [[2026-09-28-pr-007-hud-e-minimapa]]
