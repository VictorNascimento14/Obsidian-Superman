---
tipo: adr
numero: 4
data: 2026-09-28
status: aceito
tags: [adr, espaco, render, voo]
---

# ADR-004 — Espaço em escala real: cena separada e velocidade proporcional à distância

## Contexto

O pedido de 28/09 (fase 2 do [[roadmap]]) é voar até o Sol, ver a Terra e os planetas. O
jogador escolheu **escala real**, com velocidade proporcional à distância, entre duas opções:
tamanhos e distâncias de verdade, ou um sistema solar comprimido de brinquedo.

A Terra tem 6.371 km de raio; o Sol fica a 1,5×10¹¹ m. O jogo tem uma câmera com `far` de 6 km
e um buffer de profundidade comum, e a cidade é plana.

## Decisão

- **A cidade é um recorte plano sobre a Terra esférica.** O referencial da cidade é o
  geocêntrico transladado: o centro da Terra fica em (0, −R, 0). Voar reto para longe é
  física certa; só a altitude precisa ser medida até a esfera.
- **O espaço é outra cena, desenhada antes da cidade** (uma `RenderPass` própria). A cidade
  vem por cima e limpa só a profundidade. A câmera do espaço fica na origem, com a orientação
  da câmera do jogo, e cada corpo é posto relativo a ela.
- **Espaço escalado:** corpo além de 100 km é trazido para 100 km com o raio reduzido na
  mesma razão. O tamanho aparente não muda, e um buffer de profundidade comum serve de 1 m a
  10¹² m.
- **Velocidade proporcional à distância:** acima de 20 km, o boost vira hipervelocidade, com
  alvo de `1,2 × altitude`, e há um teto duro de `2,5 × altitude` em qualquer altitude. A
  subida é exponencial (25 km → 7.000 km em 8 s), e a aproximação é suave: em cada quadro o
  herói anda bem menos que a distância até a superfície, então nunca a atravessa.
- **Só Metrópolis tem chão.** A colisão da cidade vale até 3 km de altitude e 60 km de
  distância. Fora disso, o herói não desce abaixo de 30 km. O resto do planeta existe para
  ser visto, não pisado.
- **A Terra é procedural** (ADR-001): continentes, calotas, desertos, nuvens, luzes das
  cidades e atmosfera saem de um shader com ruído próprio. No lugar de Metrópolis, o globo
  desenha a ilha com o mesmo grid de ruas, para a troca entre cidade e globo casar.

## Consequências

- ✅ De 25 km a 7.000 km em 8 s, e de volta em 15 s, no e2e.
- ✅ A peça 14 (sistema solar) só acrescenta corpos: mesma cena, mesma lei de velocidade (com
  o corpo mais perto no lugar da Terra).
- ⚠️ Com o pitch limitado a 83°, subir "mirando" deriva na horizontal. Para achar a cidade
  de volta existe o marcador na tela, e a tecla de descer (C) desce na vertical.
- ⚠️ O shader do globo é caro. A passada do espaço só roda acima de 2,5 km, e as oitavas finas
  (e as luzes de cidade em pontos) só perto.
- ⚠️ Depende da [[ADR-003-origem-flutuante]]: sem ela, o herói se desmancharia longe da cidade.

Implementado no [[2026-09-28-pr-024-espaco-e-terra]].
