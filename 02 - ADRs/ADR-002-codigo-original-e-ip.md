---
tipo: adr
numero: 2
data: 2026-09-28
status: aceito
tags: [adr, licenca]
---

# ADR-002 — Código original; Spiderbench só como referência

## Contexto

O Spiderbench (github.com/xikhar/spiderbench) é publicado sob uma licença
*Source-Available, View-Only*: permite ler e rodar localmente, **proíbe**
redistribuir qualquer parte ou usá-la em outro jogo.

Superman é propriedade da DC Comics / Warner Bros. Discovery.

## Decisão

- **Nenhuma linha de código, shader ou asset do Spiderbench entra neste repo.** Ele
  serve de referência de conceito (jogo de herói em Three.js no navegador) e nada mais.
- Código deste repositório sob **MIT**, com a ressalva de que a licença cobre só o
  código original.
- **Projeto de fã, não comercial**, com aviso no README e no jogo. O emblema do
  herói é desenhado em código, estilizado, sem reproduzir logotipo oficial.

## Consequências

- ✅ Repositório público sem risco de violar a licença da referência.
- ⚠️ Se o projeto um dia virar produto, o personagem precisa ser trocado — o código
  não depende do nome (o herói é `hero`, não `superman`, nos módulos).
