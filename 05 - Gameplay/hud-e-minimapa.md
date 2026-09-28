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

## Carga solar

`setSolar(k)`: a barra dourada **CARGA SOLAR 87%** logo acima da energia da visão de calor,
só visível com carga. Os avisos **CARGA SOLAR MÁXIMA** e **CARGA SOLAR ESGOTADA** saem pelo
`toast` — ver [[carga-solar]].

## No espaço

Acima de 20 km: altitude até a superfície da Terra (perto de outro corpo, "JÚPITER a 700 km"),
velocidade em km/s (e quantas vezes a luz, que em escala real se passa), modo
**HIPERVELOCIDADE**, e marcadores na tela (`setBeacon`: nome e distância). Metrópolis (depois
da Lua, TERRA) e o Sol ficam presos à borda quando fora de vista; os outros corpos só aparecem
na tela, e dois no mesmo lugar mostram só o primeiro. Distâncias por `distance`: km até 1
milhão, depois "149,6 milhões de km" e "4,5 bilhões de km"; o objetivo das missões usa
`goalDistance` (metros perto). Ver [[espaco]].

## PRs

- [[2026-09-28-pr-007-hud-e-minimapa]]
- [[2026-09-28-pr-024-espaco-e-terra]] — altitude, km/s e marcadores no espaço
- [[2026-09-28-pr-025-sistema-solar]] — marcadores de todos os corpos, corpo mais perto, milhões e bilhões de km
- [[2026-09-28-pr-026-carga-solar]] — barra dourada CARGA SOLAR
