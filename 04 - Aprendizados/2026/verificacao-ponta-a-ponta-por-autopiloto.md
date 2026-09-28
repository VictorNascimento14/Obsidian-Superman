---
tipo: aprendizado
data: 2026-09-28
tags: [aprendizado, teste, e2e]
---

# Verificação de ponta a ponta por autopiloto

## O que é

O teste unitário prova a lógica (anel, queda, drone). Não prova que **dá para
cumprir a missão jogando**. Para isso: Chrome headless (puppeteer-core) com um
autopiloto injetado pelo gancho `window.__game` que dirige o herói como um jogador —
mira o alvo, voa, dispara — e lê o estado da missão a cada 4–5 s.

## O que ele achou (e o que não era bug)

1. **Anel atrás de prédio**: o autopiloto em linha reta travou a 99 m do anel 3 — o
   caminho reto atravessava um arranha-céu. Não é bug: o anel está acima dos telhados;
   o jogador sobe. O autopiloto ganhou "se há parede a 80 m, suba".
2. **Parado no chão**: depois do resgate o herói pousa; o autopiloto seguia mandando
   "frente" e o herói **andava**. Falta de `jump` no script, não no jogo.
3. **A mira é o centro da tela, não o yaw/pitch**: a câmera olha levemente para baixo,
   para o herói. Apontar yaw/pitch para o drone erra; o autopiloto passou a corrigir
   pelo erro entre `camera.getWorldDirection()` e o vetor câmera → drone — exatamente
   o que o jogador faz com o mouse.

## Resultado

Anéis 8/8, resgate (pega a 31 m, pouso, +150) e drones 5/5 — com a energia zerando e
travando no meio da luta, o que confirma o balanceamento da [[visao-de-calor]].

## Como evitar regressão

O harness vive no repositório a partir do PR seguinte (`scripts/e2e`).
