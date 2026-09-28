---
tipo: sistema
data: 2026-09-28
area: mundo
arquivos: [src/world/traffic.js, src/world/trafficView.js]
tags: [sistema, mundo, trafego]
---

# Tráfego e pedestres

![[trafego.jpg]]

## Carros (`traffic.js`, testado)

- **900 carros**, mão direita, **duas faixas por sentido** a 2,8 m e 8,2 m do eixo.
  Cada faixa tem velocidade fixa (11 e 15 m/s) e os carros nascem a ≥ 6 m um do outro:
  na reta ninguém ultrapassa, então não precisa de lógica de desvio. Havia um ±5% por
  carro que fazia o mais rápido atravessar o da frente (141 pares sobrepostos em 60 s →
  30). Os 30 que sobram saem da curva juntos na mesma faixa — ninguém cede a vez; o teto
  está anotado no código.
- **Cruzamento**: ao cruzar a faixa de pedestre (11 m antes do eixo), o carro lista as
  saídas abertas — reto, direita, esquerda — respeitando a borda da ilha e as ruas
  fechadas do parque (`streetOpen` da [[cidade-procedural]]). Segue reto em ~65% das
  vezes quando pode.
- **Curva**: Bézier quadrática do ponto de decisão até 11 m depois do eixo na rua
  nova, com o controle no cruzamento das duas linhas de faixa. A direção do carro é
  a derivada da curva. Sem teletransporte (o teste confere o salto máximo por passo).
- Direita na mão direita: indo +x, a direita é +z; indo +z, a direita é −x.

## Pedestres

**1.400**, cada um preso ao perímetro do próprio quarteirão, numa distância aleatória
dentro da calçada, andando a 1,1–1,7 m/s em um dos dois sentidos. Origem nos pés: a
`CapsuleGeometry` mede só o trecho reto (0,9 m + 2 × 0,22 de tampas), e eles flutuavam
26–31 cm acima da calçada.

## Visual (`trafficView.js`)

- Um `InstancedMesh` por tipo (carros, luzes dos carros, pedestres), com matrizes
  `DynamicDrawUsage` atualizadas por quadro.
- Carro = carroceria (cor por instância; **táxi amarelo é 30% da frota** — é Metropolis),
  cabine escura e rodas numa geometria mesclada com vertex colors.
- **Faróis e lanternas** em `MeshBasicMaterial` com a cor escalada por `night`
  (0,25 de dia → 3,75 à noite, HDR para o bloom).
- Pedestre sem sombra: pequeno demais para o custo.

## Números

| Item | Valor |
|---|---|
| Draw calls extras | 3 (+1 da sombra dos carros) |
| Triângulos no quadro com sombra | ~800 mil (de ~440 mil) |

✅ Resolvido no [[2026-09-28-pr-013-recorte-e-agua]]: **recorte por distância** —
carros até 900 m e pedestres até 300 m da câmera, compactados no começo do buffer.
~30% menos triângulos por quadro. A cor é copiada com `setColorAt(n, col.fromArray(...))`
— sem `subarray`, que criava uma view por instância visível a cada quadro
([[2026-09-28-pr-014-revisao-de-codigo]]).

## PRs

- [[2026-09-28-pr-009-trafego-e-pedestres]]
- [[2026-09-28-pr-013-recorte-e-agua]]
- [[2026-09-28-pr-014-revisao-de-codigo]] — cor do recorte sem alocar
- [[2026-09-28-pr-018-mundo-chao-e-pedestres]] — faixa sem ultrapassagem; pedestres no chão
