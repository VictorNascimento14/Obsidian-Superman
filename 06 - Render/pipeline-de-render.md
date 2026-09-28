---
tipo: sistema
data: 2026-09-28
area: render
arquivos: [src/render/renderer.js, src/render/sky.js, src/render/post.js, src/render/quality.js]
tags: [sistema, render]
---

# Pipeline de render

## O que faz

Desenha a cena com céu físico, sombras direcionais, mapa de ambiente e
pós-processamento (bloom, tone mapping ACES, vinheta, SMAA).

## Como funciona

1. **`renderer.js`** — `WebGLRenderer` **sem MSAA** (o SMAA do pós faz o antialias) e
   **sem tone mapping** (quem aplica a curva é o `ToneMappingEffect`; aplicar nos dois
   dobraria a curva). Sombra `PCFShadowMap`.
2. **`sky.js`** — `Sky` do three (Preetham + nuvens procedurais do r186) acompanhando a
   câmera. Uma **segunda instância dividindo o mesmo material** vive numa cena só dela,
   de onde o `PMREMGenerator` tira o `scene.environment` a cada troca de horário.
   - A **luz direcional** segue o foco numa caixa de ±260 m, com o centro arredondado
     para a grade de texel **nos eixos da própria câmera de sombra** — sem isso a sombra
     "nada" quando o herói anda. A luz fica a 2.000 m do foco (far 3.000): com o sol a
     18°, a caixa cobre ~840 m de chão na direção dele, e a 800 m esse trecho caía antes do
     `near`.
   - Abaixo do horizonte o shader do `Sky` devolve preto chapado; à **noite** ele é
     escondido e entra uma **cúpula de gradiente** (horizonte azul de poluição luminosa)
     + 1.500 estrelas. A luz direcional vira lua: elevação mínima de 18°, cor fria.
3. **`post.js`** — `postprocessing`: `RenderPass` → `EffectPass(bloom, toneMapping,
   vignette, smaa)`, buffer `HalfFloat` para o bloom enxergar valores > 1.
4. **`quality.js`** — presets `baixa | media | alta` (`?q=` na URL; só chave própria
   do objeto — `?q=constructor` abria o jogo com NaN no pixel ratio).

## Parâmetros que importam

| Parâmetro | Valor | Efeito |
|---|---|---|
| Limiar do bloom | 1.0 | Só brilho real (emissivo, sol) floresce; reflexo do céu no vidro não |
| Tone mapping | ACES filmic | AgX foi testado e dessaturou demais o dia |
| Névoa | 450 m → distância de visão do preset | Esconde a borda da cidade |
| Caixa de sombra | ±260 m | Cobre a vizinhança do herói; longe, sem sombra |
| Luz de sombra | 2.000 m do foco; near 10, far 3.000; bias −0,0002 (~0,6 m) | Sol baixo cabe inteiro no frustum |

## Horários (tecla T)

`amanhecer → dia → entardecer → noite`. Cada preset define elevação/azimute do sol,
cor/intensidade da luz, turbidez, exposição e `night` (0–1), que outros sistemas
leem para acender janelas e faróis.

## Armadilhas

- **Snap de texel é nos eixos da luz, não do mundo**: arredondar x/z desloca o shadow map
  por frações de texel (um ponto fixo variava 0,97 texel com o herói andando). Projete o
  foco em `right` e `up` da câmera de sombra, arredonde, reconstrua.
- **Bias de sombra ortográfica é em profundidade normalizada**: aumentar o `far` exige baixar
  o bias na mesma proporção (−0,0004 × 1.590 m ≈ −0,0002 × 2.990 m ≈ 0,6 m).
- **`PCFSoftShadowMap` foi removido no r186**: o three avisa e cai para `PCFShadowMap`.
- **`THREE.Clock` está depreciado**: usar `THREE.Timer` com `timer.connect(document)`
  (pausa quando a aba sai de foco).
- **`renderer.info` zera a cada `render()`**: o pós chama vários; para medir o quadro
  inteiro, `info.autoReset = false` e `info.reset()` no início do quadro.

## PRs

- [[2026-09-28-pr-002-pipeline-de-render]]
- [[2026-09-28-pr-019-render-sombras]] — luz a 2.000 m e snap nos eixos da luz
