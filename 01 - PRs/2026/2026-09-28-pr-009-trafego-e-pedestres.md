---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 9
url: https://github.com/VictorNascimento14/Superman/pull/9
branch: feat/trafego-e-pedestres
tags: [pr, mundo, trafego]
status: merged
---

# PR #9 — Tráfego e pedestres

## 🎯 Contexto

A cidade estava vazia. Carros e gente dão escala (o herói passando baixo sobre a rua)
e são a base das missões de resgate.

## 🔧 Mudanças

- `src/world/traffic.js` — simulação pura. Ver [[trafego-e-pedestres]].
- `src/world/trafficView.js` — instâncias de carros, luzes e pedestres.
- `tests/traffic.test.js` — 3 testes: determinismo; **200 carros por 90 s simulados**
  sempre na rua, dentro da ilha, fora do parque e sem salto; pedestres na calçada.
- `src/main.js` — liga tráfego e visual.

## 🧠 Decisões técnicas

- **Velocidade fixa por faixa** em vez de IA de seguir o carro da frente: resolve a
  ultrapassagem na reta de graça. Teto conhecido: na curva dois carros ainda podem se
  sobrepor por um instante.
- **Pedestre no perímetro do quarteirão**: nunca atravessa a rua, então não precisa
  de semáforo.

## ⚠️ Armadilhas e aprendizados

- **380 carros pareciam zero**: numa grade de 840 trechos de rua, dá menos de meio
  carro por trecho. Subiu para 900, com rodas de 6 lados e pedestres em low-poly.

## 🧪 Como testar

1. `npm test` — 35 testes.
2. `npm run dev`, descer até ~15 m sobre uma avenida: táxis e carros passando,
   virando nos cruzamentos; **T** até a noite: faróis e lanternas.

## 📎 Documentação afetada

- [[trafego-e-pedestres]]
- [[2026]] (changelog)
