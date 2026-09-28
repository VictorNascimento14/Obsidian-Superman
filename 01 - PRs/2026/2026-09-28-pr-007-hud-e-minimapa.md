---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 7
url: https://github.com/VictorNascimento14/Superman/pull/7
branch: feat/hud-minimapa
tags: [pr, gameplay, hud]
status: merged
---

# PR #7 — HUD com velocímetro e minimapa

## 🎯 Contexto

Voar a 1.500 km/h sem número na tela não tem graça, e as missões que vêm a seguir
precisam de objetivo e de marcador no mapa.

## 🔧 Mudanças

- `src/ui/hud.js` — HUD e minimapa. Ver [[hud-e-minimapa]].
- `src/main.js` — HUD aparece com o jogo e some na pausa; **H** abre os controles;
  "BARREIRA DO SOM" no estrondo; marcador do Planeta Diário.

## 🧠 Decisões técnicas

- **DOM em vez de desenhar o HUD no WebGL**: texto nítido, CSS responsivo, zero custo
  de GPU. O que muda todo quadro (minimapa) vai num canvas 2D.
- **Mapa-base pré-desenhado**: 2.064 retângulos uma vez, não a cada quadro.

## 🧪 Como testar

1. `npm run dev`, clicar e voar: velocidade e altitude mudam, o mapa gira com o olhar.
2. **Shift** segurado: selo SUPERSÔNICO e "BARREIRA DO SOM".
3. **H** abre e fecha os controles.

Também conferido: o deploy do [[2026-09-28-pr-006-deploy-pages]] roda no Pages com
console limpo (screenshot headless da URL pública).

## 📎 Documentação afetada

- [[hud-e-minimapa]]
- [[2026]] (changelog)
