---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 16
url: https://github.com/VictorNascimento14/Superman/pull/16
branch: fix/pausa-e-pointer-lock
tags: [pr, input, audio, pausa]
status: merged
---

# PR #16 — Pausa: despausar só com o pointer lock confirmado

## 🎯 Contexto

Achados da revisão do [[2026-09-28-pr-014-revisao-de-codigo]] no fluxo de pausa. O mais
grave apareceu em duas fatias da revisão, cada uma chegando a ele por conta própria:
voltar logo depois do Esc deixava o jogo rodando sem mouse, sem jeito de travar de novo.

## 🔧 Mudanças

- `src/main.js` — o clique no overlay libera o som e **pede** o lock. O jogo despausa no
  `pointerlockchange`, com o lock confirmado. `start()` continua existindo para o
  autopiloto e o e2e (que jogam sem lock). Pausar suspende o áudio.
- `src/core/input.js`:
  - `lock()` trata a rejeição e devolve `false` quando o navegador não tem pointer lock
    (aí o jogo despausa direto, só com o teclado);
  - o `blur` limpa os botões do mouse, além das teclas;
  - Espaço só arma o pulo com o ponteiro travado.
- `src/audio/audio.js` — `suspend()`.

## 🧠 Decisões técnicas

- **Despausar no evento, não no clique.** O `pointerlockchange` é a única confirmação de
  que o lock pegou. Na recusa, nada muda de estado, então o overlay simplesmente fica.
- **`audio.start()` continua no clique**: a política de autoplay exige gesto do usuário,
  e o `pointerlockchange` não conta como gesto.
- **Suspender na pausa em vez de ouvir `visibilitychange`**: trocar de aba sempre solta o
  lock, e portanto sempre pausa. Um ponto só cobre Esc, Alt+Tab e troca de aba.
- **Pulo só com o lock**: segurar Espaço continua decolando pelo `up`; só a borda (`jump`)
  que ficava armada durante a pausa deixou de existir.

## Verificação

Clique real no overlay (puppeteer, gesto confiável), com a recusa do Chrome simulada:

| Caso | Antes | Depois |
|---|---|---|
| Lock recusado | overlay some, jogo roda sem mouse, `SecurityError` sem tratamento | overlay fica, jogo pausado, console limpo |
| Lock aceito | despausa | despausa |

## ⚠️ Armadilhas e aprendizados

- [[pointer-lock-recusado-apos-esc]] — o Chrome recusa o relock por ~1 s depois do Esc e
  não dispara `pointerlockchange`.
- O e2e não pegaria isso: ele chama `__game.start()` e nunca passa pelo lock.

## 🧪 Como testar

1. `npm test` (43), `npm run build` e `npm run e2e` (anéis 1 · resgate 1 · drones 1, 921 pontos).
2. `npm run dev`: aperte Esc e clique em "Clique para voar" na hora. O overlay continua; um
   segundo clique, um instante depois, volta ao jogo com o mouse funcionando.
3. Segure o botão direito (visão de calor), dê Alt+Tab e volte: o feixe não fica ligado.

## 📎 Documentação afetada

- [[voo-e-camera]], [[audio]]
- [[pointer-lock-recusado-apos-esc]]
- [[2026]] (changelog)
