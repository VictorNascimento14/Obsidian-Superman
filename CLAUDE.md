# CLAUDE.md — cofre Obsidian-Superman

Instruções para qualquer agente que escreva neste cofre.

## Regras

- **A raiz só tem `README.md` e `CLAUDE.md`.** Nota nova vai numa pasta 00–10.
- **Link interno é `[[wiki-link]]`**, nunca link markdown para outra nota.
- **Toda nota tem frontmatter** com ao menos `tipo`, `data` e `tags`.
- **Nome de arquivo em kebab-case**, sem acento. Nota de PR:
  `01 - PRs/2026/AAAA-MM-DD-pr-NNN-slug.md`.
- **ADR reserva número antes de nascer**: incremente `proximo_numero_livre` em
  [[adrs]] no mesmo commit que cria o arquivo.
- **PR mergeado = nota do PR + entrada em `03 - Changelog/2026.md`**, no mesmo
  commit do cofre, com as duas leituras ("Para o jogador" / "Para o time técnico").
- **Mecânica, sistema de render ou sistema de mundo novo** ganha nota própria em
  `05`, `06` ou `07` e entrada no MOC [[arquitetura]].
- **Divergência entre cofre e código** se corrige no mesmo PR que a descobriu.
- Commits do cofre: `docs(pr-NNN): <título curto>`. Sem menção a ferramenta de IA
  em commit, branch ou nota.

## Se mudou X, documente em Y

| Mudou | Onde |
|---|---|
| Qualquer PR mergeado | `01 - PRs/2026/` + `03 - Changelog/2026.md` |
| Decisão difícil de reverter | `02 - ADRs/ADR-NNN-slug.md` |
| Mecânica de jogo / controle | `05 - Gameplay/` |
| Renderização, shader, pós | `06 - Render/` |
| Cidade, colisão, tráfego | `07 - Mundo/` |
| Build, CI, deploy | `08 - Infra e Deploy/` |
| Armadilha que custou > 30 min | `04 - Aprendizados/2026/` |
