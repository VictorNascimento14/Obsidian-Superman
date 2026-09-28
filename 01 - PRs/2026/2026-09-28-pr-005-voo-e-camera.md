---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 5
url: https://github.com/VictorNascimento14/Superman/pull/5
branch: feat/voo
tags: [pr, gameplay, voo]
status: merged
---

# PR #5 — Física de voo, câmera de perseguição e controles

## 🎯 Contexto

O verbo central do jogo. Princípio 3 da [[visao-de-produto]]: sensação de voo antes
de gráfico.

## 🔧 Mudanças

- `src/player/flight.js` — física pura (9 testes). Ver [[voo-e-camera]].
- `src/player/camera.js` — terceira pessoa com colisão, FOV e altura por velocidade, tremor.
- `src/core/input.js` — teclado + mouse com pointer lock; setas giram sem mouse.
- `src/fx/shockwave.js` — anel do estrondo sônico.
- `src/ui/overlay.js` — tela de início/pausa com controles e aviso de fã.
- `src/main.js` — reescrito: o jogo de verdade no lugar da vitrine; nasce na sacada
  do Planeta Diário. Gancho `autopilot` para dirigir o herói em teste headless.
- `src/world/layout.js` — caixa do globo a 75% do raio.

## 🧠 Decisões técnicas

- **Velocidade persegue alvo** em vez de força/massa: controle previsível, inércia
  ajustável por marcha.
- **Sem gravidade no ar**: o herói voa; cair seria frustrante e fora do personagem.
- **Câmera sem lerp de posição**: única forma de não perder o herói a 400 m/s.

## ⚠️ Armadilhas e aprendizados

- [[heroi-sumido-em-supervelocidade]] — enquadramento, não render.
- Estrondo sônico medido tarde demais; orientação parada com o jogo pausado; nascer
  de frente para o globo. Detalhes em [[voo-e-camera]].

## 🧪 Como testar

1. `npm test` — 29 testes.
2. `npm run dev`, clique, **Espaço** decola, **W** voa, **Shift** segurado → supersônico
   (anel + tremor), **C** até encostar num telhado → pousa.

## 📎 Documentação afetada

- [[voo-e-camera]]
- [[heroi-sumido-em-supervelocidade]]
- [[2026]] (changelog)
