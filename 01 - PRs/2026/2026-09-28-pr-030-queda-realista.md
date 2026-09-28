---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 30
url: https://github.com/VictorNascimento14/Superman/pull/30
branch: feat/queda-realista
tags: [pr, destruicao, mundo]
status: merged
---

# PR #30 — Prédios caindo de verdade

## 🎯 Contexto

Peça 19 da fase 3 do [[roadmap]]: "a animação realmente dos prédios caindo ao atravessar eles".
Antes, a parte de cima descia quase reta (tombava até ~8°) e afundava na rua, e só caía quando
o dano de uma faixa somava 50%. Na prática, prédio largo nunca caía. O jogador escolheu:
supersônico (ou com carga solar) derruba a parte de cima de qualquer prédio, e mais devagar
cedem só os andares em volta do furo.

## 🔧 Mudanças

- `src/world/damage.js` — `DAMAGE.topple`: força ≥ 340 derruba em qualquer largura.
- `src/world/fall.js` (novo, puro): o bloco desce esmagando o toco e tomba sobre a borda da
  base do lado do golpe; em 1,4 s se parte em segmentos que caem como corpos rígidos, afundam
  um quarto no que atingiram e se desfazem. O toco é esmagado pelo segmento de baixo.
- `src/world/collapse.js`:
  - até 5 segmentos (`makePart` de faixas);
  - chão = toco dentro da pegada, rua ou telhado vizinho fora;
  - impacto com pedaços grandes e nuvem de poeira em anel;
  - evento `cede` 0,3–0,65 s depois de um furo que não derruba.
- `src/world/city.js` — `makePart(bi, yFrom, yTo)` com tampas de concreto nos cortes.
- `src/world/interior.js` — laje em placas de 6 m; `interiors.collapseRegion` derruba tudo
  numa esfera, laje inclusive.
- `src/main.js`:
  - `cede` derruba o interior em volta e abre um rombo de vários andares;
  - estalo e tremor na quebra;
  - o dano vem antes do interior, e prédio que vai desabar não monta interior.
- `src/fx/breach.js` — pool de poeira de 160 para 480.
- Testes: 4 em `fall.test.js` e 1 de dano. Total 112. O e2e ganhou **derrubar largo** e
  **ceder**, e o **desabar** espera 12 s.

## 🧠 Decisões técnicas

- **Tombo sobre a borda da base, do lado do golpe.** É o que faz a parte de cima cair na
  direção em que o herói passou, em vez de descer reto. O giro acelera com o ângulo, como uma
  árvore caindo.
- **Segmentos com tampa.** Cada segmento é uma cópia exata do trecho (as mesmas janelas no
  mesmo lugar), e a tampa de concreto no corte evita a casca oca à vista.
- **Afundar antes de se desfazer.** Sumir no instante do toque parecia um corte de cena; um
  quarto da altura enterrado no chão ou no telhado lê como impacto.
- **O dano decide antes do interior.** No supersônico, montar o interior e desmontar no mesmo
  quadro (o prédio já ia cair) custava quadros.

## Verificação

![[queda-realista.jpg]]

Em cima: a 420 m/s, a torre pega fogo na base, tomba, se parte em segmentos no ar e cai atrás
do vizinho. Embaixo: a 150 m/s num prédio de 82 m, a explosão, e os andares em volta do furo
cedendo, até o rombo de vários andares.

- e2e:
  - atravessar: 682 pedaços de entulho, 161 peças quebradas lá dentro;
  - desabar: caiu, toco de 3 m;
  - derrubar largo: prédio de 82 m a 420 m/s desabou, com segmentos no ar (de vários
    prédios, porque o herói supersônico furou uns seis em 1,8 s);
  - ceder: prédio de 81 m a 150 m/s ficou em pé, e os andares cederam.
- Vendo um desabamento: 16,8 ms por quadro (p95 16,7). Numa passada supersônica pelo centro
  (34 rupturas em 6 s), 17,5 ms de média e quadros de 33 ms: o custo é da GPU (poeira grande
  sobreposta), não da CPU (a quebra lá dentro custa 0,6 ms por quadro).

## ⚠️ Armadilhas e aprendizados

- **Compilação de shader no primeiro uso** aparece como pico de 67–133 ms na primeira
  explosão, no primeiro interior e no primeiro segmento. Medir desempenho exige aquecer antes.
- **Ordem da atualização do esmagamento**: atualizado antes da checagem de impacto, o toco
  não era marcado como esmagado quando o segmento de baixo era o último a se desfazer.
- **Câmera de screenshot em cânion de rua** não vê o prédio caindo: ela vai para cima dos
  telhados, na diagonal.

## 🧪 Como testar

1. `npm test` (112) e `npm run build`. `npm run e2e`: tudo de antes, mais **derrubar largo**
   e **ceder**.
2. `npm run dev`: segure Shift até o SUPERSÔNICO e atravesse um prédio alto. Ele cai, tombando
   para o lado para onde você foi. Em cruzeiro, fure um prédio largo: os andares em volta do
   furo cedem.

## 📎 Documentação afetada

- [[destruicao]], [[interior-dos-predios]], [[runbook-e2e]]
- [[roadmap]], [[2026]] (changelog)
