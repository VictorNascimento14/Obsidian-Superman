---
tipo: moc
data: 2026-09-28
tags: [moc, produto]
---

# Visão de produto

**Superman — Metropolis Flight** é um jogo de navegador em terceira pessoa em que o
jogador voa sobre uma Metropolis procedural. A inspiração de formato é o
*Spiderbench* (jogo de balanço de teia do Homem-Aranha em Three.js), mas o verbo
central é outro: **voar**, não balançar. Nada do código ou dos assets do
Spiderbench é reaproveitado — a licença dele é só leitura. Ver [[ADR-002-codigo-original-e-ip]].

## O que o jogador faz

- **Voa** livremente: decolar, pairar, acelerar, supervelocidade com estrondo sônico.
- **Usa poderes**: visão de calor (derrete e explode alvos), sopro congelante.
- **Resgata e combate**: missões espalhadas pela cidade — pessoas em queda,
  carros em perigo, drones hostis.
- **Explora** uma cidade com arranha-céus, ruas, tráfego, parque, baía e um globo
  dourado no topo do prédio do Planeta Diário.

## Princípios

1. **Roda no navegador comum.** WebGL2, 60 fps em GPU integrada com o preset médio.
2. **Tudo procedural.** Sem Blender nesta máquina e sem asset baixado: modelos,
   texturas e cidade saem de código. Ver [[ADR-001-stack-threejs-vite-procedural]].
3. **Sensação de voo antes de gráfico.** A física e a câmera são o produto.
