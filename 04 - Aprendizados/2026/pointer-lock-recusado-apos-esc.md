---
tipo: aprendizado
data: 2026-09-28
tags: [aprendizado, input, navegador]
---

# Pointer lock recusado logo depois do Esc

## Sintoma

O jogador aperta Esc, o overlay de pausa aparece e ele clica em "Clique para voar" logo
em seguida. O jogo volta a rodar, mas o mouse não mira mais, o Esc não pausa e o overlay
nunca volta: só recarregando a página. O console mostra um `SecurityError` sem
tratamento.

## Causa

O Chrome recusa um `requestPointerLock()` feito até cerca de 1 s depois de uma saída por
Esc. A Promise rejeita, dispara `pointerlockerror` e **não** dispara
`pointerlockchange`. O código despausava no próprio clique, sem esperar o lock:

```js
createOverlay(() => { input.lock(); start(); }); // start() roda mesmo se o lock falhar
```

Sem lock, `mousemove` é ignorado. E como nada mudou de estado, nenhum evento traz o
overlay de volta.

## O que fazer

- **Despausar no `pointerlockchange`, com `document.pointerLockElement` confirmado.** O
  clique só *pede* o lock. Se o lock for recusado, o overlay continua e o próximo clique
  tenta de novo.
- **O que exige gesto do usuário continua no clique**: o `audio.start()` (política de
  autoplay) fica no handler do botão, não no evento de lock.
- **Tratar a rejeição**: `Promise.resolve(el.requestPointerLock()).catch(() => {})`. O
  `Promise.resolve` cobre navegadores antigos, em que a função devolve `undefined`.
- **Sem pointer lock no navegador, despausar direto**: o jogo continua jogável só com o
  teclado.
- **Testar com clique real**: no puppeteer, `page.click()` é gesto confiável. Para simular
  a recusa, sobrescreva `HTMLCanvasElement.prototype.requestPointerLock` com
  `evaluateOnNewDocument`.

Visto no [[2026-09-28-pr-016-pausa-e-pointer-lock]].
