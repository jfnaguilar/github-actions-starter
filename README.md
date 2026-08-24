# GitHub Actions Starter Kit

Conjunto de workflows prontos para deixar qualquer repositório mais robusto: CI, segurança, qualidade de PR, release e Docker.

Todos os workflows **se autodetectam**: se o repositório não tem Python, o CI de Python não roda; sem `package.json`, o CI de Node não roda; sem `Dockerfile`, o workflow de Docker não roda. Dá para copiar a pasta `.github/` inteira em qualquer projeto sem quebrar nada.

## Instalação

Copie a pasta `.github/` para a raiz do seu projeto:

```bash
git clone https://github.com/jfnaguilar/github-actions-starter.git /tmp/gha-kit
cp -r /tmp/gha-kit/.github SEU_PROJETO/
cd SEU_PROJETO
git add .github && git commit -m "ci: adiciona workflows do starter kit" && git push
```

Ou copie só os arquivos que interessam — cada workflow é independente.

## O que vem junto

| Arquivo | O que faz | Quando roda |
|---|---|---|
| `workflows/ci-python.yml` | ruff (lint + format), mypy, pytest com cobertura em Python 3.10–3.13 + Windows/macOS, build de sdist/wheel | push, PR |
| `workflows/ci-node.yml` | detecta npm/pnpm/yarn, roda lint, typecheck, build e test em Node 20/22/24, `npm audit` | push, PR |
| `workflows/codeql.yml` | análise estática de segurança do GitHub, detectando as linguagens do repo | push, PR, toda segunda 06:00 UTC |
| `workflows/security.yml` | Gitleaks (segredos), dependency review em PRs, Trivy no filesystem | push, PR, toda segunda 07:00 UTC |
| `workflows/pr-quality.yml` | valida título Conventional Commits, rótulos automáticos, rótulo de tamanho, actionlint | PR |
| `workflows/release.yml` | changelog automático desde a tag anterior + release com artefatos | ao criar tag `v*.*.*` |
| `workflows/docker.yml` | hadolint, build multi-arquitetura, push para GHCR, varredura Trivy da imagem | push em main/tags, PR |
| `workflows/stale.yml` | marca e fecha issues/PRs abandonados | diariamente |
| `dependabot.yml` | atualiza actions, pip, npm e Docker semanalmente | semanal |
| `labeler.yml` | regras de rótulo por caminho de arquivo | usado pelo `pr-quality.yml` |
| `pull_request_template.md`, `ISSUE_TEMPLATE/` | modelos de PR e issue | — |

## Configuração necessária no repositório

Nada de secrets extras: tudo usa o `GITHUB_TOKEN` automático. Mas vale ajustar:

1. **Settings → Actions → General → Workflow permissions**: deixe em *Read repository contents and packages permissions* (os workflows pedem permissão extra só onde precisam).
2. **Settings → Code security**: ligue *Dependency graph*, *Dependabot alerts* e *Code scanning* — CodeQL e Trivy publicam resultados ali (em repositório público é grátis; em privado exige GitHub Advanced Security, e nesse caso os passos de SARIF simplesmente falham sem quebrar o build).
3. **Rótulos**: crie os rótulos usados pelo labeler (`documentação`, `ci`, `python`, `javascript`, `docker`, `testes`, `dependências`) e os de tamanho (`size/XS` … `size/XL`). O `sync-labels` só aplica rótulos já existentes.
4. **Proteção de branch** (Settings → Rules → Rulesets): exija os checks `Lint & format (ruff)`, `Testes (Python 3.12)` ou `Node 22` como obrigatórios antes do merge — é isso que transforma o CI em rede de segurança de verdade.

## Convenções que o kit assume

- **Conventional Commits** nos títulos de PR: `feat:`, `fix:`, `docs:`, `refactor:`, `perf:`, `test:`, `ci:`, `chore:`.
- **Tags semânticas** `v1.2.3` para disparar release.
- Em Python: `ruff` como linter/formatador e `pytest` para testes.
- Em Node: scripts `lint`, `build`, `test`, `typecheck` no `package.json` (os que não existirem são pulados).

## Ajustes comuns

- **Trocar as versões da matriz**: edite `strategy.matrix` no arquivo de CI correspondente.
- **Deixar o mypy obrigatório**: remova `continue-on-error: true` do job `typecheck` em `ci-python.yml`.
- **Não publicar imagem Docker**: apague `workflows/docker.yml` ou troque `push:` para `false`.
- **Desativar o stale bot**: apague `workflows/stale.yml` (ele fecha issues antigas — nem todo projeto quer isso).
