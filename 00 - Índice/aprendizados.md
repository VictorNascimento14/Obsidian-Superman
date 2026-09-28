---
tipo: moc
data: 2026-09-28
tags: [moc, aprendizado]
---

# Aprendizados

Bugs instrutivos e armadilhas, do mais recente para o mais antigo.

- [[perfilar-alocacao-antes-de-cortar]] — some a alocação por quadro e perfile com o lixo coletado: o maior alocador não estava na lista
- [[pointer-lock-recusado-apos-esc]] — despause no `pointerlockchange`, não no clique: o Chrome recusa o relock logo após o Esc
- [[remover-da-cena-nao-libera-gpu]] — `scene.remove` não descarta geometria/material; meça o patamar de `renderer.info.memory`
- [[fps-headless-nao-mede-ganho]] — declare o trabalho removido (triângulos), não o FPS do headless

- [[verificacao-ponta-a-ponta-por-autopiloto]] — autopiloto no navegador prova que a missão é cumprível

- [[heroi-sumido-em-supervelocidade]] — projete o objeto na tela antes de caçar bug de render
