---
tipo: sistema
data: 2026-09-28
area: gameplay
arquivos: [src/game/missions.js, src/game/missionLogic.js]
tags: [sistema, missoes, gameplay]
---

# Missões

![[missoes.jpg]]

Rodízio fixo: **treino de voo → resgate → drones → …**, com 4 s de respiro entre
elas. **N** pula a atual (conta como falha). Pontos e missões cumpridas aparecem no
topo durante o respiro.

## Os três tipos

| Missão | Objetivo | Falha | Pontos |
|---|---|---|---|
| **Treino de voo** | atravessar 8 anéis em ordem | tempo (80 s, +6 s por anel) | 25 por anel + 100 + 2×segundos que sobraram |
| **Resgate** | pegar no ar quem cai de um arranha-céu e pousar | a pessoa chega ao chão | 150 |
| **Drones hostis** | abater 5 drones com a [[visao-de-calor]] | tempo (120 s) | 200 + segundos que sobraram |

## Como funciona

**Lógica pura (`missionLogic.js`, 5 testes)**

- `ringCrossed(p0, p1, anel)` — o herói **troca de lado do plano** do anel entre dois
  quadros e o ponto de cruzamento está dentro do raio. A 400 m/s ele anda 7 m por
  quadro e nunca "está" dentro do anel; testar distância não funcionaria.
- `makeRingCourse` — anéis 170–260 m um do outro, curvas de até ±45°, dentro da
  ilha, com o anel **inteiro** acima do telhado mais alto num raio em volta, 12–60 m
  de folga. A normal aponta para o próximo. O caminho *entre* anéis pode cruzar prédio
  — é o desafio. Se a partida não comporta nenhum anel (mar, beirada olhando para fora),
  recomeça de dentro da ilha rumo ao centro: o circuito vazio congelava o jogo.
- `stepFaller` — gravidade com arrasto quadrático, terminal 55 m/s.
- `tryCatch` — raio de 4,5 m em volta do herói.
- `burnDrone` — vida 1, 0,8/s de fogo (~1,25 s por drone).

**Visual e fluxo (`missions.js`)**

- Anéis: toros dourados; o próximo brilha em HDR e gira, os passados somem.
- Resgate: prédio de um nível só (sem recuo onde a pessoa bateria antes do chão), a
  300–900 m do herói; a pessoa acena na beirada e **cai** quando o herói chega a
  300 m ou depois de 14 s. Pega, ela vai "nos braços" até o pouso.
- Drones: 5 em órbita num cruzamento, a 45–85 m de altura e sempre 20 m acima do
  telhado mais alto num raio de 40 m (senão atravessavam prédios); o olho vermelho segue
  o herói e o corpo esquenta com o dano; registrados em `heatVision.targets`; explodem ao morrer.
- **Pilar de luz** de 700 m na cor da missão, e marcadores no [[hud-e-minimapa]].
- **Fim de missão** descarta geometria e material de tudo que ela criou (`disposeTree`);
  o `ringGeo` é compartilhado e fica — ver [[remover-da-cena-nao-libera-gpu]].

## Verificação de ponta a ponta

As três foram **concluídas por autopiloto no navegador headless** — ver
[[verificacao-ponta-a-ponta-por-autopiloto]].

## PRs

- [[2026-09-28-pr-010-missoes]]
- [[2026-09-28-pr-014-revisao-de-codigo]] — dispose no fim da missão
- [[2026-09-28-pr-015-missoes-borda-e-drones]] — anéis longe da ilha, drones acima dos prédios
