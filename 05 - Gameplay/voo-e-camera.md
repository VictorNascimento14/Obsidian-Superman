---
tipo: sistema
data: 2026-09-28
area: gameplay
arquivos: [src/player/flight.js, src/player/camera.js, src/core/input.js, src/fx/shockwave.js, src/ui/overlay.js]
tags: [sistema, voo, camera, controles]
---

# Voo, câmera e controles

![[voo.jpg]]

## Controles

| Tecla | No ar | No chão |
|---|---|---|
| Mouse | direção do voo (olhar) | olhar |
| W / S | frente / trás na direção do olhar (3D) | andar |
| A / D | lateral | lateral |
| Espaço | subir | decolar |
| C ou Q | descer (pousa ao encostar) | — |
| Shift | boost; **segurar 1,2 s em frente = supersônico** | correr |
| T | hora do dia | |
| Esc | pausa (solta o mouse) | |

Ctrl ficou de fora de propósito: Ctrl+W fecha a aba.

**Pausa e retomada** (`main.js`, `input.js`): o clique em "Clique para voar" só *pede* o
pointer lock; o jogo despausa no `pointerlockchange`, quando o lock pega de fato. O
Chrome recusa o relock por ~1 s depois de sair com Esc — o overlay continua e o próximo
clique tenta de novo (ver [[pointer-lock-recusado-apos-esc]]). Trocar de janela limpa
teclas **e** botões do mouse; Espaço só arma o pulo com o ponteiro travado.

## Como funciona (`flight.js`, testado)

- **Duas fases**: `ground` (anda relativo ao yaw, sola a `FLIGHT.footDepth` do telhado) e
  `air` (sem gravidade — é o Superman).
- **Três marchas**: cruzeiro 48 m/s, boost 150, supersônico 420. A velocidade
  **persegue** o alvo por `lerp` exponencial com taxa por marcha (2,2 / 1,4 / 0,9 1/s):
  quanto mais rápido, mais inércia.
- **Pairar**: sem comando, a velocidade cai por `e^(−2,8·dt)` — ele freia sozinho.
- **Colisão subdividida**: o passo é quebrado em pedaços de ≤ 0,8 m (máx. 60) e a
  componente da velocidade que entra em **cada** superfície tocada é removida, uma de
  cada vez (desliza na parede). Cortar contra a soma das normais (chão + parede = uma
  diagonal) lançava o herói parede acima ao bater rente à rua.
  Impacto acima de 70 m/s vira evento → tremor de câmera.
- **Espaço**: acima de 20 km o boost vira hipervelocidade (alvo 1,2 × altitude) e há um
  teto duro de 2,5 × altitude — sobe exponencial, desce suave sem atravessar. Fora do mundo
  plano não há colisão, e o piso é 30 km: ver [[espaco]].
- **Atravessar prédio**: voando, ≥ 30 m/s para dentro da parede fura em vez de parar — ver
  [[destruicao]]. Sem pouso enquanto estiver dentro de um prédio.
- **Pouso**: tocou chão/telhado a menos de 18 m/s, sem subir → `ground`.
  Andar para fora da beirada volta para `air` (não cai).
- **Eventos**: `takeoff`, `land`, `supersonic`, `sonicboom` (cruzou 340 m/s — anel
  de choque em `fx/shockwave.js`), `impact`.
- **Orientação**: de pé, cabeça para cima e peito para onde anda; acima de 8–24 m/s
  mistura para "cabeça na direção da velocidade, peito para baixo", com **rolagem**
  proporcional à taxa de giro do yaw.

## Câmera (`camera.js`)

- Posição **recalculada todo quadro** a partir do herói, sem `lerp`: a 400 m/s
  qualquer atraso deixa o herói fora do quadro. Só distância, FOV e tremor suavizam.
- Distância 5 m no chão, 6,5 → 9 m no ar conforme a velocidade; **altura sobe**
  (+1,2 → +4,4 m) e o **FOV abre** (62° → 76°).
- Raio **do herói** (não do ombro) até a câmera: se bate em prédio, a câmera encurta em
  vez de atravessar. Saindo do ombro, encostado numa parede à direita, o raio nascia
  dentro do prédio — e o raycast ignora a caixa que contém a origem.

## Armadilhas

- **A câmera é filha do grupo `world`** (origem flutuante, [[ADR-003-origem-flutuante]]):
  a posição dela é verdadeira, mas o `lookAt` quer o alvo em espaço de render —
  `camera.parent.localToWorld(alvo)`.

- **"Antes" do estrondo medido depois da aceleração** nunca via a travessia: o lerp é
  que cruza os 340 m/s. Mede no início do `update`.
- **Herói "sumido" em supervelocidade** — ver [[heroi-sumido-em-supervelocidade]].
- **Com o jogo pausado a orientação não roda**: nascia de lado. `updateOrientation(10)`
  na criação.
- **Nascer olhando para o globo** fazia o primeiro "W" pousar em cima da caixa dele;
  a caixa do globo caiu para 75% do raio.

## PRs

- [[2026-09-28-pr-005-voo-e-camera]]
- [[2026-09-28-pr-016-pausa-e-pointer-lock]] — despausar só com o lock confirmado
- [[2026-09-28-pr-017-voo-colisao-e-camera]] — corte por superfície; câmera traça do herói
- [[2026-09-28-pr-021-atravessar-predios]] — atravessar prédios
- [[2026-09-28-pr-023-origem-flutuante]] — câmera filha do mundo
- [[2026-09-28-pr-024-espaco-e-terra]] — hipervelocidade, teto e piso no espaço
