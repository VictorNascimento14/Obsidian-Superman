---
tipo: pr
data: 2026-09-28
projeto: Superman
pr: 31
url: https://github.com/VictorNascimento14/Superman/pull/31
branch: feat/visao-corta
tags: [pr, poderes, visao-de-calor, destruicao]
status: merged
---

# PR #31 — A visão de calor corta e quebra prédios

## 🎯 Contexto

Peça 20, a última da fase 3 do [[roadmap]]: "a visão de raio laser deve também cortar os
prédios e quebrar". Antes, o raio só deixava marcas escuras de queimado.

## 🔧 Mudanças

- `src/powers/cuts.js` (novo, puro):
  - `createCuts().hit` queima células de 1 m por prédio, face e faixa de 8 m (as duas faixas na
    divisa); 70% da largura queimada fatia;
  - o raio parado num ponto (0,6 s, ou 0,15 s carregado) estoura um furo;
  - `faceOf` dá a face, a coordenada e a largura pela normal do acerto.
- `src/powers/heatvision.js` — brilho de brasa aditivo sobre as marcas recentes (esfria em
  2,5 s); `surfaceHit`.
- `src/world/collapse.js` — `slice(bi, y, dir)`: corte limpo; o toco fica na altura do corte,
  e a queda não esmaga.
- `src/world/fall.js` — `createFall(..., { crushG, tiltAccel })`: no corte limpo, sem descida
  e com tombo mais rápido.
- `src/main.js`:
  - raio em fachada (não em telhado): o corte fatia (aviso CORTADO!);
  - o furo estourado entra no jogo como uma ruptura sintética a 120 m/s, com furo aberto,
    explosão, interior e andares cedendo.
- Testes: 6 em `cuts.test.js` e 1 de queda. Total 119. O e2e ganhou **cortar** e **quebrar**.

## 🧠 Decisões técnicas

- **O furo do raio é uma ruptura sintética.** Reaproveita tudo o que a passagem do herói já
  faz — abertura de verdade, interior, explosão, dano e os andares cedendo — sem outro caminho
  no código.
- **Corte limpo não esmaga.** Na travessia, o golpe destrói um andar, e a parte de cima desce
  esmagando os de baixo. No corte, os andares de baixo estão inteiros: a parte de cima só tomba
  sobre a borda do corte e escorrega para fora, deixando o toco liso.
- **70% da largura, não 100%.** As pontas da fachada ficam em ângulo rasante para quem está na
  rua, e cortar até a quina era quase impossível. Com 70% cortado, o que sobra não segura.
- **Divisa de faixas queima as duas.** Ao varrer uma parede, a mira sobe e desce com a distância;
  um corte na divisa se dividia entre duas faixas e nenhuma chegava ao limite.

## Verificação

![[visao-corta.jpg]]

Em cima: varrendo a fachada, a linha em brasa; o prédio fatiado; o toco liso com escombros. Embaixo:
com a mira parada, o furo estourando, os pedaços, e o prédio desabando depois de dois furos.

- e2e:
  - cortar: varrendo a fachada de 38 m em 2,4 s, o prédio foi **fatiado**;
  - quebrar: com o raio parado 1 s num ponto, **1 furo** estourou;
  - o resto passou como antes (derrubar largo, ceder, atravessar a Terra).

## ⚠️ Armadilhas e aprendizados

- **Cobertura de 79% com o limiar em 85%**: medida no navegador, a varredura de ±54° a 11 m da
  parede cobre ±15 m de uma fachada de 38,5 m.
- **Telhado não é fachada**: acerto com normal para cima contaria como corte numa face; o corte
  exige normal quase horizontal.

## 🧪 Como testar

1. `npm test` (119) e `npm run build`. `npm run e2e`: tudo de antes, mais **cortar** e
   **quebrar**.
2. `npm run dev`: pare na rua de frente para um prédio, segure F (ou o botão direito) e varra a
   mira de um lado ao outro da fachada. Segure o raio parado num ponto para estourar um furo.

## 📎 Documentação afetada

- [[visao-de-calor]], [[destruicao]], [[runbook-e2e]]
- [[roadmap]] (fase 3 concluída), [[arquitetura]], [[2026]] (changelog)
