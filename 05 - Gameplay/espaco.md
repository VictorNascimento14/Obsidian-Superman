---
tipo: sistema
data: 2026-09-28
area: gameplay
arquivos: [src/space/nav.js, src/space/bodies.js, src/space/space.js, src/player/flight.js, src/render/sky.js, src/render/post.js, src/ui/hud.js, src/main.js]
tags: [sistema, espaco, voo, render]
---

# Espaço e o sistema solar

![[espaco-subida.jpg]]

![[sistema-solar.jpg]]

## O que é

Da cidade, o herói sobe até o espaço em escala real: o céu escurece, a neblina afina, a
cidade vira a ilha desenhada no globo, e a Terra aparece inteira, com continentes, nuvens,
oceano brilhando ao sol, luzes das cidades no lado da noite e o halo da atmosfera. Na volta,
um marcador na tela aponta Metrópolis. Decisões em [[ADR-004-espaco-em-escala-real]].

Dali dá para ir a qualquer corpo do sistema solar, todos em tamanho e distância de verdade:
o Sol (a 25 s da cidade), a Lua, Mercúrio, Vênus, Marte, Júpiter, Saturno com os anéis,
Urano e Netuno (a uns 35 s). Marcadores na tela mostram onde cada um está.

## Como funciona

**Navegação** (`space/nav.js`, lógica pura testada):

- a cidade é um recorte plano sobre a Terra, com o centro dela em (0, −R, 0);
- `altitude(p)` mede até a esfera; `fromCity(p)` mede o arco sobre a superfície;
- `nearCity(p)` é o mundo plano (< 3 km de altitude e < 60 km da cidade), onde valem
  colisão e pouso.

- `nearestSurface(bodies, p)` acha o corpo com a **superfície** mais perto (não o centro)
  e a distância até ela. É esse corpo que comanda o voo.

**Sistema solar** (`space/bodies.js`, lógica pura testada):

- `solarSystem(sunDir)` devolve Terra, Sol, Lua e os 7 planetas no referencial da cidade. O
  Sol fica a 1 UA na direção em que o céu o mostra (`sky.state.sunTrue`), e a hora do dia
  gira o sistema inteiro em volta da Terra (é a Terra girando);
- os planetas ficam no plano da eclíptica, cada um no raio de órbita real, em ângulos fixos
  escolhidos para aparecerem em direções diferentes do céu da Terra: Mercúrio e Vênus perto
  do Sol; Marte, Saturno e Urano de noite. Numa sessão de jogo eles quase não andariam, então
  o sistema é estático;
- o eixo de cada planeta é o "para cima" da cidade — que é o da câmera, em qualquer lugar —
  inclinado pela inclinação real dele: as faixas ficam deitadas na tela, os anéis de Saturno
  abrem 27° para quem vem da Terra, e Urano (98°) gira de lado. A Lua mostra sempre a mesma
  face para a Terra;
- `sunlight(bodies, p)` diz quanto do Sol chega em p. Conta com o tamanho aparente dos dois
  discos: a sombra tem penumbra, e a Terra vista de Marte não faz sombra nenhuma.

**Voo** (`flight.js`):

- acima de 20 km do corpo mais perto, o boost vira hipervelocidade: o alvo é
  `1,2 × distância` até a superfície dele;
- o teto duro é `max(420 m/s, 2,5 × distância)`. Em cada quadro o herói anda bem menos que a
  distância até o corpo mais perto, então nunca atravessa a superfície de nada;
- fora do mundo plano não há colisão. Todo corpo tem piso: 30 km na Terra (fora da cidade) e
  `max(20 km, 1% do raio)` nos outros (7.000 km no Sol, 700 km em Júpiter). Só Metrópolis
  tem chão;
- `flight.altitude` (até a Terra) vai para a transição; `flight.nearest` (corpo e distância)
  vai para o HUD.

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

**Espaço escalado** (`space/space.js`): uma cena à parte, com a câmera na origem. Até 100 km
o corpo fica onde está; além disso a distância cresce com o log
(`100 km × (1 + ln(d / 100 km) / 2)`), com o raio reduzido na mesma razão. O tamanho
aparente não muda, e quem está mais longe continua atrás: é a ordem que decide quem passa na
frente de quem. Nenhum corpo fica menor que ~2 px (Netuno a 4,5 bilhões de km ainda é um
ponto no céu, e a Terra vista de Netuno é um pálido ponto azul).

**Globo** da Terra. O shader tem ruído próprio (value noise com hash aritmético, sem `sin`):

- continentes em fbm, desertos por latitude e calotas;
- oceano com brilho especular do sol, nuvens que andam;
- luzes das cidades: regiões povoadas de longe, pontos de perto;
- borda de atmosfera e um halo numa casca maior;
- Metrópolis é a ilha de 2,3 km com o **mesmo grid de ruas** e o parque no lugar certo. As
  ruas somem por `fwidth` quando ficam menores que um pixel;
- em volta dela há mar, com a borda recortada por ruído (uma baía);
- o globo é iluminado pelo **Sol verdadeiro** (`sky.state.sunTrue`), não pela luz da
  cidade, que à noite vira lua a 18°.

**Sol**:

- granulação que ferve devagar, manchas em latitudes médias e o limbo mais escuro e mais
  vermelho;
- o brilho depende do tamanho na tela, como o olho que se ajusta: pequeno, é HDR alto e o
  bloom faz a estrela; grande, é baixo, e aparece a superfície laranja com as células;
- a coroa é uma casca de faces de trás (5 raios). Cada pixel mede o quanto o raio de visão
  passa perto do centro e brilha mais quanto mais perto. Funciona de qualquer distância, até
  de dentro dela: rente à superfície, o céu acima do horizonte fica alaranjado.

**Planetas e Lua**: um shader com quatro tipos (define `KIND`, cada um paga só o que usa):

| Tipo | Corpos | O que desenha |
|---|---|---|
| Rochoso | Lua, Mercúrio | terras altas e mares escuros (na Lua, só na face voltada para a Terra), crateras |
| Nuvens | Vênus | faixas largas quase sem contraste |
| Marte | Marte | poeira, planícies escuras, calotas polares |
| Gigante gasoso | Júpiter, Saturno, Urano, Netuno | faixas de latitude torcidas por turbulência; a Grande Mancha Vermelha e a Mancha Escura de Netuno |

Os anéis de Saturno ficam no plano do equador: anel C, anel B brilhante, a divisão de Cassini,
anel A, e a sombra do planeta neles. Cada corpo é iluminado pela direção dele ao Sol.

**Marcadores** (`main.js`): Metrópolis (depois da Lua vira TERRA) e o Sol ficam presos à
borda quando estão fora de vista; os outros corpos só aparecem quando estão na tela. Dois
marcadores no mesmo lugar (a Lua vista do Sol cai em cima da Terra) mostram só o primeiro.
Distância legível: km até 1 milhão, depois "149,6 milhões de km", "4,5 bilhões de km".

**Luz do herói**: no espaço, a luz direcional é o Sol visto de onde o herói está, e some na
sombra da Terra (ou de qualquer corpo entre ele e o Sol).

**A hora do dia** (T) gira o sistema inteiro em volta da Terra. Longe dela (além de
100.000 km), T não muda nada: tiraria o planeta de perto do herói.

## Parâmetros que importam

| Parâmetro | Valor | Por quê |
|---|---|---|
| `SPACE.from` | 20 km | Abaixo disso, o voo de sempre |
| `SPACE.hyper` | 1,2 × altitude | 25 km → 7.000 km em 8 s |
| `SPACE.cap` | 2,5 × altitude | Aproximação suave, nunca atravessa |
| `SPACE.flatRadius` / `floor` | 60 km / 30 km | Só Metrópolis tem chão |
| Espaço escalado | 100 km | Profundidade comum de 1 m a 10¹² m |
| Detalhe fino do globo | < 4.000 km | De longe, oitava fina só serrilharia |
| Piso dos outros corpos | `max(20 km, 1% do raio)` | Sol a 7.000 km, Júpiter a 700 km |
| Espaço escalado além de 100 km | log, `LOG_K` = 2 | Ordem de profundidade entre os corpos |
| Raio aparente mínimo | 0,9 px | Planeta longe ainda é um ponto |
| `near` / `far` da câmera do espaço | 10 m / 6.000 km (escalados) | Cabe a coroa vista de dentro |

**Tempos de viagem** com boost, medidos no voo puro, partindo de 20.000 km de altitude (da
cidade até lá são mais ~10 s):

| Destino | Tempo |
|---|---|
| Lua | 6 s |
| Sol | 14 s |
| Mercúrio · Vênus · Marte | 16 a 18 s |
| Júpiter · Saturno · Urano · Netuno | 20 a 23 s |

A velocidade máxima no caminho para Netuno passa de 6.000 vezes a da luz.

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
- **O chão da cidade só existe no mundo plano.** Júpiter fica abaixo do plano da cidade
  (y < 0), e duas coisas liam o chão dela sem perguntar se o herói estava perto: o teste de
  "câmera dentro de prédio" ligava a poeira na tela inteira (uma névoa marrom), e o som
  ambiente da cidade tocava no máximo. As duas agora exigem `nearCity`.
- **Todos os corpos a 100 km não têm ordem.** Com um corpo só, trazer tudo para 100 km
  servia. Com vários, a Lua e a Terra caíam na mesma distância e a profundidade decidia no
  acaso. O log preserva a ordem.
- **A coroa vista de dentro passava do `far`.** Rente ao Sol, a casca de 5 raios chegava a
  3.200 km escalados, e o `far` de 2.000 km a recortava: um polígono preto no céu laranja. O
  `far` subiu para 6.000 km; a precisão da profundidade quem decide é o `near`.
- **Planeta acima de 1 no HDR vira névoa.** O bloom pega o que passa de 1 e o espalha pela
  tela inteira. O lado do dia dos planetas fica abaixo disso; só o Sol passa.
- **O eixo pela eclíptica deixava as faixas em pé.** Os horários do céu põem o Sol a até 48°
  de altura, e o plano que contém a linha Terra–Sol fica, no mínimo, inclinado assim em relação
  ao horizonte da cidade. Como a câmera usa sempre esse horizonte, Júpiter aparecia com as
  faixas quase na vertical. O eixo dos planetas passou a partir do "para cima" da cidade.

## PRs

- [[2026-09-28-pr-024-espaco-e-terra]]
- [[2026-09-28-pr-025-sistema-solar]] — Sol, Lua e planetas, corpo mais perto, espaço
  escalado em log, marcadores
