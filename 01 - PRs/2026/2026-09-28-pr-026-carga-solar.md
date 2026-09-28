---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 26
url: https://github.com/VictorNascimento14/Superman/pull/26
branch: feat/carga-solar
tags: [pr, poderes, espaco, voo, destruicao, hud]
status: merged
---

# PR #26 — Carregar de Sol e ficar muito mais forte

## 🎯 Contexto

Peça 15 da fase 2 do [[roadmap]]: "voar até o Sol, que é onde eu ficarei super hiper mega
forte". O jogador escolheu uma carga que dura minutos e cai devagar, para dar tempo de voltar
e usar. É a condição da peça 16: a visão de calor carregada atravessa a Terra.

## 🔧 Mudanças

- `src/powers/solar.js` (novo, puro):
  - `solarFlux` (inverso do quadrado × eclipse);
  - `createSolar().update(dt, flux, firing)` (enche, esvazia, avisa `'full'`/`'empty'`);
  - `SOLAR` com os multiplicadores.
- `src/player/flight.js` — `flight.charge`:
  - velocidades e teto × `1 + 1,5 × carga`;
  - contra prédio, `might = 1 + 2 × carga` divide o freio da fachada e dos metros, e o
    evento de ruptura leva `force`.
- `src/world/collapse.js` — o dano usa `force` (com a carga, maior); `damageAt` exposto.
- `src/powers/heatvision.js` e `energy.js` — `setCharge`: alcance, dano, reserva paga pelo
  Sol, raio mais grosso e branco-dourado.
- `src/player/hero.js` — `setCharge`: brilho de borda (fresnel) no emissivo do corpo e da
  capa, e um halo que pulsa.
- `src/ui/hud.js` — barra dourada **CARGA SOLAR**.
- `src/main.js` — a carga a cada quadro (com o eclipse de `sunlight`) e os avisos.
- Testes:
  - 4 em `solar.test.js` (fluxo, encher em ~5 s, durar 4 min, equilíbrio a ~7 raios);
  - 1 de energia (o Sol paga o disparo);
  - 2 de voo (prédio freia menos com força 3×; voo 2,5×).
  - Total 90.
- e2e: cenário **carga solar**. O `__launch` passou a escolher só prédio intacto, com
  largura mínima.

## 🧠 Decisões técnicas

- **Inverso do quadrado de verdade.** Encher depende de chegar perto (7 raios do centro só
  empatam), e a Terra, a 1 UA, não carrega nada, sem precisar de uma "zona" arbitrária.
- **Força separada da velocidade no evento de ruptura.** O dano do prédio usa `force`; o
  entulho continua voando com a velocidade de verdade.
- **A velocidade para furar não muda.** Baixá-la com a carga faria o herói afundar no
  telhado ao pousar.
- **Aura de borda, não emissivo uniforme.** O uniforme desbotava o traje de dia.

## Verificação

![[carga-solar.jpg]]

Da esquerda para a direita: sem carga; carregado de dia (a silhueta acende, o traje fica
azul); disparando carregado (raio grosso, branco-dourado); carregado à noite (o halo).

- e2e: a 80.000 km do Sol, a carga encheu em **5,6 s** (aviso CARGA SOLAR MÁXIMA). De volta
  à cidade com 100%, a 150 m/s o prédio de 34 m **caiu de uma vez**; sem carga, o mesmo golpe
  daria 0,31 de dano e só furaria.
- Voo puro: entrando a 150 m/s num prédio, sem carga sobram 67% da velocidade em 40 m;
  carregado, entrando a 375 m/s, sobram 85%.

## ⚠️ Armadilhas e aprendizados

Registradas em [[carga-solar]]:

- emissivo uniforme desbota o herói;
- para comparar o freio de um prédio, a direção do voo não pode ajudar ninguém: o amortecimento
  de pairar e o alvo do boost confundiam a medida;
- a velocidade de furar fica igual.

## 🧪 Como testar

1. `npm test` (90) e `npm run build`. `npm run e2e`: tudo de antes, mais **carga solar**.
2. `npm run dev`: voe até o ◇ SOL e fique rente a ele. A barra **CARGA SOLAR** enche em
   segundos, e o herói brilha. Volte a Metrópolis (◇ TERRA, depois ◇ METRÓPOLIS): com a carga,
   voe contra um prédio e ele cai; a visão de calor (botão direito / F) sai grossa e dourada.

## 📎 Documentação afetada

- [[carga-solar]] (nova)
- [[visao-de-calor]], [[destruicao]], [[voo-e-camera]], [[heroi-e-capa]], [[hud-e-minimapa]],
  [[runbook-e2e]]
- [[roadmap]], [[arquitetura]], [[2026]] (changelog)
