---
tipo: sistema
data: 2026-09-28
area: gameplay
arquivos: [src/powers/solar.js, src/player/flight.js, src/powers/heatvision.js, src/powers/energy.js, src/world/collapse.js, src/player/hero.js, src/ui/hud.js, src/main.js]
tags: [sistema, poderes, espaco, sol]
---

# Carga solar

![[carga-solar.jpg]]

## O que faz

Perto do Sol o herói se enche de energia (a barra dourada **CARGA SOLAR**). Carregado, ele:

- voa até 2,5 vezes mais rápido;
- bate nos prédios com o triplo da força: o prédio o freia menos, e um golpe que antes só
  furava derruba o prédio;
- dispara uma visão de calor que vai 6 vezes mais longe, queima 5 vezes mais e não gasta a
  reserva, com o raio mais grosso e branco-dourado;
- brilha: a silhueta irradia dourado, dentro de um halo.

A carga dura minutos longe do Sol (escolha do jogador). É a condição da peça 16: a visão de
calor carregada atravessa a Terra.

## Como funciona

**Carga** (`powers/solar.js`, lógica pura testada):

- `solarFlux(dist, lit)`: o fluxo do Sol pela lei do inverso do quadrado, em "superfícies do
  Sol" (1 rente a ela, 2×10⁻⁵ na Terra), vezes a parte do disco que chega (`sunlight`, do
  eclipse: atrás de um planeta não carrega);
- `update(dt, flux, firing)`: soma `fill × fluxo − drain`, menos `beam` enquanto a visão de
  calor dispara. Devolve `'full'` e `'empty'` para os avisos **CARGA SOLAR MÁXIMA** e
  **CARGA SOLAR ESGOTADA**;
- rente à superfície, enche em ~5 s. A ~7 raios do centro (4,8 milhões de km), o que entra
  empata com o que sai. Na Terra, não enche nada.

**Efeitos** (`flight.charge` e `heatVision.setCharge`, a cada quadro no `main.js`):

- **Voo**: os três níveis de velocidade (cruzeiro, boost, supersônico) e o teto no mundo
  plano × `1 + 1,5 × carga`. No espaço, a hipervelocidade continua mandando: o alvo é o maior
  dos dois.
- **Contra prédio**: `might = 1 + 2 × carga`. A fachada e cada metro freiam `might` vezes
  menos, e o evento de ruptura leva `force = velocidade × might`, que o dano do prédio usa no
  lugar da velocidade (o entulho continua usando a velocidade de verdade). A velocidade para
  furar (30 m/s) não muda: carregado, o herói ainda pousa num telhado sem afundar.
- **Visão de calor**: alcance × `1 + 5 × carga`, dano nos alvos × `1 + 4 × carga`, e a
  reserva drena × `1 − carga` (cheia, de graça).
- **Visual** (`hero.setCharge`): brilho de borda (fresnel) no emissivo dos materiais do herói
  e da capa, que o bloom espalha, e um halo aditivo que pulsa.

## Parâmetros que importam

| Parâmetro | Valor | Efeito |
|---|---|---|
| `SOLAR.fill` | 0,2/s × fluxo | Cheia em ~5 s rente ao Sol |
| `SOLAR.drain` | 1/240 por s | Cheia dura 4 minutos |
| `SOLAR.beam` | 1/45 por s a mais | Disparando sem parar, a carga cheia dura ~38 s |
| `SOLAR.speed` | 1,5 | Voo × 2,5 com a carga cheia |
| `SOLAR.force` | 2 | Força contra prédio × 3 |
| `SOLAR.range` | 5 | Visão de calor alcança 4,2 km |
| `SOLAR.power` | 4 | Dano × 5 |

Medido no voo puro: entrando a 150 m/s num prédio, sem carga sobram 67% da velocidade depois
de 40 m; carregado, entrando a 375 m/s, sobram 85%.

## Armadilhas

- **Emissivo por igual desbota o herói.** A primeira versão somava dourado em toda a
  superfície: de dia o traje azul virou branco-rosado, sem sombra. O brilho de borda
  (`pow(1 − n·v, 4)`) deixa o traje com a cor dele e acende só a silhueta.
- **Comparar freio de prédio com o herói parado no comando confunde.** Sem comando, o
  amortecimento de pairar (2,8/s) freia mais que o prédio; com boost, a direção puxa cada um
  para o próprio alvo. O teste entra cada um na velocidade do próprio boost: a direção não
  ajuda ninguém, e o que sobra é o freio do prédio.
- **Baixar a velocidade de furar com a carga** faria o herói atravessar o telhado ao pousar.
  Ela fica em 30 m/s.

## PRs

- [[2026-09-28-pr-026-carga-solar]]
