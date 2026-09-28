---
tipo: moc
data: 2026-09-28
tags: [moc, arquitetura]
---

# Arquitetura

Mapa dos sistemas do jogo. Cada linha vira link quando a nota do sistema nascer.

## Gameplay (`05 - Gameplay`)

- [[heroi-e-capa]] — modelo, poses e capa simulada
- [[voo-e-camera]] — física de voo, controles e câmera
- [[hud-e-minimapa]] — velocímetro, objetivo, avisos e minimapa

## Render (`06 - Render`)

- [[pipeline-de-render]] — renderer, céu, luz, sombra, pós e horários

## Mundo (`07 - Mundo`)

- [[cidade-procedural]] — layout, prédios, parque, marco, texturas
- [[colisao]] — esfera, raio e altura do telhado

## Infra (`08 - Infra e Deploy`)

- [[build-e-ci]] — dev server, testes, build e os dois workflows
- [[github-pages]] — deploy a cada merge; [[runbook-conferir-deploy]]
