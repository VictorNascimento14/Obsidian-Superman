---
tipo: sistema
data: 2026-09-28
area: gameplay
arquivos: [src/space/nav.js, src/space/space.js, src/player/flight.js, src/render/sky.js, src/render/post.js, src/ui/hud.js]
tags: [sistema, espaco, voo, render]
---

# Espaço e a Terra

![[espaco-subida.jpg]]

## O que é

Da cidade, o herói sobe até o espaço em escala real: o céu escurece, a neblina afina, a
cidade vira a ilha desenhada no globo, e a Terra aparece inteira, com continentes, nuvens,
oceano brilhando ao sol, luzes das cidades no lado da noite e o halo da atmosfera. Na volta,
um marcador na tela aponta Metrópolis. Decisões em [[ADR-004-espaco-em-escala-real]].

## Como funciona

**Navegação** (`space/nav.js`, lógica pura testada):

- a cidade é um recorte plano sobre a Terra, com o centro dela em (0, −R, 0);
- `altitude(p)` mede até a esfera; `fromCity(p)` mede o arco sobre a superfície;
- `nearCity(p)` é o mundo plano (< 3 km de altitude e < 60 km da cidade), onde valem
  colisão e pouso.

**Voo** (`flight.js`):

- acima de 20 km, o boost vira hipervelocidade: o alvo é `1,2 × altitude`;
- o teto duro é `max(420 m/s, 2,5 × altitude)`;
- fora do mundo plano não há colisão, e há um piso a 30 km;
- `flight.altitude` vai para o HUD e para a transição.

**Transição por altitude** (`main.js`):

| Altitude | O que acontece |
|---|---|
| 2,5 km | liga a passada do espaço (a Terra por baixo) |
| 2,5 → 4 km | o céu abaixo do horizonte fica transparente |
| 4 km | a cidade de verdade sai; entra a ilha desenhada no globo |
| 4 → 40 km | o céu inteiro esmaece, a luz azul e o reflexo do céu apagam, o vento some |
| 20 km | hipervelocidade, HUD em km e km/s, marcador de Metrópolis |

A neblina afina com a altitude: foi pensada para olhar na horizontal, e de cima a cidade
tem de aparecer.

**Globo** (`space/space.js`): uma cena à parte, em espaço escalado (corpo além de 100 km
vem para 100 km com o raio na mesma razão). O shader tem ruído próprio (value noise com hash
aritmético, sem `sin`):

- continentes em fbm, desertos por latitude e calotas;
- oceano com brilho especular do sol, nuvens que andam;
- luzes das cidades: regiões povoadas de longe, pontos de perto;
- borda de atmosfera e um halo numa casca maior;
- Metrópolis é a ilha de 2,3 km com o **mesmo grid de ruas** e o parque no lugar certo. As
  ruas somem por `fwidth` quando ficam menores que um pixel;
- em volta dela há mar, com a borda recortada por ruído (uma baía);
- o globo é iluminado pelo **Sol verdadeiro** (`sky.state.sunTrue`), não pela luz da
  cidade, que à noite vira lua a 18°.

## Parâmetros que importam

| Parâmetro | Valor | Por quê |
|---|---|---|
| `SPACE.from` | 20 km | Abaixo disso, o voo de sempre |
| `SPACE.hyper` | 1,2 × altitude | 25 km → 7.000 km em 8 s |
| `SPACE.cap` | 2,5 × altitude | Aproximação suave, nunca atravessa |
| `SPACE.flatRadius` / `floor` | 60 km / 30 km | Só Metrópolis tem chão |
| Espaço escalado | 100 km | Profundidade comum de 1 m a 10¹² m |
| Detalhe fino do globo | < 4.000 km | De longe, oitava fina só serrilharia |

## Armadilhas

- **A luz da cidade não é o Sol.** À noite ela vira lua com elevação mínima de 18°, e o globo
  aparecia iluminado como de dia. O globo usa o `sunPosition` do céu, que tem a elevação real.
- **O chão plano da cidade do outro lado da Terra** puxaria o herói para y = 0. Fora do mundo
  plano, a colisão nem roda.
- **Céu Preetham abaixo do horizonte é branco**: a 10 km a tela sumia num véu. O céu de baixo
  esmaece mais cedo que o de cima.
- **"Atrás da câmera" se testa antes da projeção**: depois dela, um ponto além do `far`
  também sai com z > 1, e o marcador de Metrópolis ia para a borda com a cidade no meio da
  tela.
- **Declaração fora de ordem no `main.js`**: o espaço precisa do layout (o parque), e o pós,
  do espaço. Um `layout` usado antes de existir travou a carga (TDZ), e só apareceu capturando
  `pageerror`.

## PRs

- [[2026-09-28-pr-024-espaco-e-terra]]
