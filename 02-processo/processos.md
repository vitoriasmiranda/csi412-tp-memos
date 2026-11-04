# Análise da Evolução e do Processo de Desenvolvimento do Memos

> Todas as evidências se referem ao repositório [usememos/memos](https://github.com/usememos/memos), analisado na tag `v0.31.0`. Os hashes de commit podem ser conferidos com `git show <hash>`.

## 1. Histórico e evolução do projeto

O Memos evoluiu de forma incremental, com mudanças registradas publicamente em commits, issues, pull requests e releases. Destacamos seis acontecimentos.

### 1.1 Criação do produto mínimo — dezembro de 2021

O primeiro commit (`2f72bfa`, 08/12/2021, "init project") já continha:

- gerenciamento de usuários (`api/user.go`, `store/user.go`);
- autenticação (`api/auth.go`);
- criação e gerenciamento de memos (`api/memo.go`, `store/memo.go`);
- persistência em SQLite (`store/sqlite.go`, `resources/sqlite.sql`).

No mesmo dia o frontend foi adicionado (`06bffd0`, "migrate frontend") e, quatro dias depois, um Dockerfile (`f70ad68`, 12/12/2021), facilitando a implantação.

---

### 1.2 Remoção da API legada (2024) e migração para Connect RPC (2025)

São dois momentos diferentes da evolução da API:

1. **Abril/maio de 2024:** a API antiga foi retirada (`755d5b8`, 12/04/2024, "retire legacy setting api"; `bbd206e`, 13/04/2024, "retire legacy api"). A API baseada em Protocol Buffers passou a ser a única, e a mudança foi publicada na release `v0.22.0` (18/05/2024).
2. **Dezembro de 2025:** a comunicação foi migrada para **Connect RPC** (PR #5338, `edd3ced`, 11/12/2025), mantendo a exposição REST via gRPC-Gateway.

**Consequências:**

- fim da duplicação entre duas APIs;
- contratos centralizados em Protocol Buffers (`proto/api/v1/`);
- geração automática de código de cliente e servidor;
- integrações que usavam a API antiga precisaram ser adaptadas (mudança incompatível).

---

### 1.3 MCP: criação, remoção e recriação a partir do OpenAPI — fevereiro a julho de 2026

A integração com MCP (Model Context Protocol), que permite que assistentes de IA acessem o Memos, passou por retrabalho:

| Data | Acontecimento | Evidência |
|------|---------------|-----------|
| 24/02/2026 | Primeiro servidor MCP, com autenticação por token pessoal (PAT) | `47d9414` |
| 05/06/2026 | Esse servidor MCP é removido | `2a4638b` |
| 09/06/2026 | Novo MCP gerado a partir da especificação OpenAPI da API | PR #6026 (`777d227`) |
| 26/07/2026 | A nova integração chega a uma release | `v0.30.0` |

Em vez de manter um servidor MCP escrito à mão, o projeto passou a **reutilizar o contrato da API** já existente (OpenAPI) para gerar as ferramentas MCP. Isso reduz duplicação e mantém o MCP sincronizado com a API.

---

### 1.4 Reconstrução do editor — junho de 2026

O commit `5a73d7d` (28/06/2026, "rebuild the editor on CodeMirror as decorated source") reconstruiu o editor sobre o CodeMirror 6 (dependências `@codemirror/*` em `web/package.json`).

O editor passou a usar *fonte decorada*: o Markdown continua visível, mas títulos, tags, links e menções recebem estilo e comportamento durante a edição.

- **Evolução perfectiva:** melhora a experiência de escrita;
- **Reuso:** usa uma biblioteca de editor consolidada em vez de código próprio;
- **Modernização:** troca um subsistema central sem abandonar o formato Markdown.

---

### 1.5 Spaces, colaboração e portabilidade — agosto e setembro de 2026

O PR #6228 (`6da8461`, 23/08/2026) introduziu múltiplos **Spaces**:

- criação de Spaces e participação de membros;
- papéis de administrador e usuário;
- visibilidade `SPACE`, com regras de autorização próprias (`core/access/memo.go`);
- migrações para SQLite, MySQL e PostgreSQL.

O design foi documentado antes em `docs/design/multi-spaces.md`.

Em seguida, o PR #6334 (`019ca26`, 16/09/2026) implementou **exportação e importação de memos em ZIP**, com conteúdo em Markdown, manifesto JSON e anexos (formato descrito em `docs/design/memos-export-format.md`).

---

### 1.6 Mudança no processo de release — setembro de 2026

Logo após a `v0.31.0`, o PR #6362 (`fa12d84`, 22/09/2026) trocou o versionamento semântico `v0.x` por **versionamento por calendário** (`26.10.0`, `26.11.0`), removeu o Release Please e reiniciou a base de migrações do banco. O workflow `.github/workflows/release.yml` passou a aceitar apenas tags no novo formato.

Esse acontecimento mostra que o **processo** também evolui, e não só o produto.

---

## 2. Processo de desenvolvimento e práticas observadas

O Memos é desenvolvido publicamente no GitHub, com issues, branches, commits, pull requests e integração contínua. Abaixo, as práticas observadas, com evidência e com os limites de cada uma.

### 2.1 Issues estruturadas

Há formulários distintos para defeitos e para novas funcionalidades (`.github/ISSUE_TEMPLATE/bug_report.yml` e `feature_request.yml`). Eles pedem problema observado, comportamento esperado, passos para reproduzir, versão utilizada e motivação.

Além disso, o workflow `.github/workflows/stale.yml` marca como inativas as issues e PRs sem atividade por 30 dias e as fecha após mais 7. Isso mantém a lista de issues enxuta, mas também pode fechar pedidos legítimos que ninguém respondeu.

### 2.2 Pull requests e revisão

Funcionalidades grandes (MCP, Spaces, exportação) entraram por pull request, o que dá rastreabilidade entre problema, implementação e testes. O arquivo `CODEOWNERS` atribui todo o repositório ao grupo `@usememos/moderators`.

**Limites observados:**

- Nos últimos 12 meses (nov/2025 a nov/2026) houve cerca de 1.015 commits, dos quais só cerca de 360 referenciam um PR na mensagem. A maior parte das mudanças vai direto para a branch principal.
- A autoria é concentrada em poucas contas, o que indica um projeto mantido por um núcleo pequeno.

**Como é a revisão na prática.** Verificamos as páginas dos dois PRs mais importantes do período:

| PR | Aberto por | Revisores listados | Mesclado por |
|----|-----------|--------------------|--------------|
| [#6228](https://github.com/usememos/memos/pull/6228) (Spaces) | `bluedbird` | `coderabbitai[bot]` (comentou) e `johnnyjoygh` (revisão solicitada, ainda pendente) | `boojack`, em 23/08 |
| [#6334](https://github.com/usememos/memos/pull/6334) (exportação) | `bluedbird` | `coderabbitai[bot]` (comentou) e `johnnyjoygh` (revisão solicitada, ainda pendente) | `boojack`, em 16/09 |

Ou seja: nos dois casos a única revisão registrada foi a de uma **ferramenta automática de revisão por IA** (CodeRabbit). O revisor humano foi indicado, mas a revisão aparece como pendente, e o PR foi mesclado por um mantenedor mesmo assim. O projeto tem a prática de revisão **formalmente**, mas na prática ela depende muito de automação e da confiança entre poucos mantenedores. Em compensação, as descrições dos PRs são detalhadas: o #6228, por exemplo, tem seções de resumo, limites de design (*Design boundaries*) e verificação.

### 2.3 Integração contínua

O projeto usa GitHub Actions (`.github/workflows/`):

| Workflow | O que verifica |
|----------|----------------|
| `backend-tests.yml` | `go mod tidy`, `golangci-lint` (análise estática) e testes Go divididos em grupos; os testes de `store` rodam em SQLite, MySQL e PostgreSQL |
| `frontend-tests.yml` | `pnpm lint`, `pnpm test` e `pnpm build` |
| `proto-linter.yml` | validação dos arquivos Protocol Buffers |
| `upgrade-smoke.yml` | instalação limpa e atualização a partir da versão estável anterior |

Na `v0.31.0` existem 187 arquivos de teste Go (`*_test.go`).

### 2.4 Commits convencionais

As mensagens usam prefixos padronizados, o que facilita ler o histórico e gerar notas de versão:

| Prefixo | Finalidade |
|---------|------------|
| `feat` | nova funcionalidade |
| `fix` | correção de defeito |
| `refactor` | refatoração |
| `test` | testes |
| `docs` | documentação |
| `chore` | manutenção |

### 2.5 Releases automatizadas

Entre abril/2026 (PR #5842) e a `v0.31.0`, o projeto usou o **Release Please** para preparar versões e atualizar changelogs a partir dos commits convencionais. Depois da `v0.31.0` a ferramenta foi removida no PR #6362, junto com a adoção do versionamento por calendário (ver 1.6).

### 2.6 Documentos de design

O projeto não usa ADRs no formato clássico, mas mantém **documentos de design** em `docs/design/` (por exemplo `multi-spaces.md`, `memos-export-format.md`, `api-abuse-controls.md`). Eles registram objetivo, decisões e consequências; `multi-spaces.md`, por exemplo, tem as seções *Non-goals* e *Alternatives considered*. Isso mostra que mudanças grandes são planejadas antes de implementadas.

### 2.7 Testes de migração

Como o Memos suporta SQLite, MySQL e PostgreSQL (`store/db/sqlite`, `store/db/mysql`, `store/db/postgres`), mudanças no banco vêm acompanhadas de migrações para os três, e o workflow `upgrade-smoke.yml` testa a atualização entre versões. Isso reduz o risco de perda de dados ao atualizar.

---

## 3. Relação com os processos de software estudados

O Memos não segue integralmente nenhum modelo. Ele combina práticas de alguns e claramente não adota outros.

### 3.1 Desenvolvimento incremental (presente)

O sistema não foi construído de uma vez. O histórico mostra entregas sucessivas, cada uma utilizável:

1. autenticação, memos e persistência em SQLite (2021);
2. interface web e Docker (2021);
3. API única em Protocol Buffers (2024) e Connect RPC (2025);
4. MCP (2026);
5. novo editor (2026);
6. Spaces e exportação/importação (2026).

Cada release acrescenta funcionalidade sobre a anterior, o que é a ideia central do desenvolvimento incremental.

### 3.2 Extreme Programming — XP (parcial)

Práticas de XP **presentes**:

- **Integração contínua:** cada push e PR passa pelos workflows da seção 2.3.
- **Testes automatizados:** 187 arquivos de teste Go, mais testes do frontend.
- **Refatoração:** commits `refactor:` frequentes, como `5a73d7d` (editor) e a migração para Connect RPC.

Práticas de XP **ausentes ou contrariadas**:

- **Pequenas entregas:** o PR #6228 (Spaces) alterou 187 arquivos (+17.369 / −2.385 linhas) e o #6334 alterou 43 arquivos (+6.507 / −721). São entregas grandes, não pequenas.
- **Programação em par** e **cliente presente:** não há evidência no repositório.
- **TDD:** existem muitos testes, mas não é possível afirmar pelo histórico que eles foram escritos antes do código.

Portanto, o projeto adota **algumas práticas técnicas** do XP, sem seguir o XP como processo.

### 3.3 Reuso de software (presente)

O Memos se apoia em componentes existentes em vez de reimplementá-los:

| Componente reutilizado | Uso | Evidência |
|------------------------|-----|-----------|
| React | interface web | `web/package.json` |
| CodeMirror 6 | editor | `@codemirror/*` em `web/package.json` |
| highlight.js | destaque de código | `web/src/components/MemoContent/highlight.ts` |
| Echo | servidor HTTP | `github.com/labstack/echo/v5` em `go.mod` |
| Connect RPC e gRPC-Gateway | API RPC e REST | `go.mod` |
| Protocol Buffers | contratos da API | `proto/api/v1/` |
| AWS SDK (S3) | armazenamento de anexos | `go.mod` |
| Especificação OpenAPI | geração das ferramentas MCP | PR #6026 |

O caso do MCP (seção 1.3) é o exemplo mais claro: o projeto descartou uma implementação própria e passou a reutilizar o contrato da API.

### 3.4 Modelos que o projeto não segue

- **Scrum:** não há sprints, backlog priorizado público, Product Owner nem Scrum Master visíveis. As entregas são contínuas, não em ciclos fixos.
- **Modelo Spotify:** não há squads, tribos ou guildas. O núcleo de mantenedores é pequeno.
- **Cascata:** não há fases sequenciais. Requisitos, implementação e testes acontecem juntos a cada funcionalidade, embora os documentos de design (2.6) tragam um planejamento prévio para mudanças grandes.