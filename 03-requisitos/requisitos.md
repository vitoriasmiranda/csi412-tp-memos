# Requisitos do Memos

> Evidências conferidas na tag `v0.31.0` do [usememos/memos](https://github.com/usememos/memos/tree/v0.31.0). Números de linha valem para essa tag.

## 4.1 Requisitos funcionais

### RF01 — Criar, editar e visualizar memos em Markdown
O sistema deve permitir criar, editar e visualizar notas (memos) escritas em Markdown.

**Evidências:**
- Editor: `web/src/components/MemoEditor/index.tsx`, baseado em CodeMirror 6 (dependências `@codemirror/*` em `web/package.json`).
- Commit `5a73d7d` (28/06/2026): reconstrução do editor como "fonte decorada", mantendo o Markdown visível.
- Destaque de sintaxe em blocos de código: `web/src/components/MemoContent/highlight.ts` (usa `highlight.js`).

### RF02 — Organizar e filtrar memos por tags
O sistema deve extrair tags escritas no texto (ex.: `#arquitetura`) e permitir filtrar memos por elas.

**Evidências:**
- Extração das tags no backend: `markdown/extensions/tag.go`.
- Campo `tags` do memo, marcado como `OUTPUT_ONLY` (calculado pelo servidor a partir do conteúdo): `proto/api/v1/memo_service.proto`, linha 244.
- Filtro por tag na listagem (ex.: `"work" in tags`): mesmo arquivo, linha 346.
- Interface: lista de tags no menu lateral (`web/src/components/AppSidebar/TagsSection.tsx`) e autocompletar de tags no editor (`web/src/components/MemoEditor/Editor/tagAutocomplete.ts`).

### RF03 — Definir a visibilidade de cada memo
O sistema deve permitir definir quem pode ler cada memo: só o autor (`PRIVATE`), usuários autenticados (`PROTECTED`), público (`PUBLIC`) ou membros de um Space (`SPACE`).

**Evidências:**
- Enum `Visibility`: `proto/api/v1/memo_service.proto`, linhas 153–163.
- Regras de leitura: função `CheckMemoReadContext` em `core/access/memo.go`.
- PR #6228: introdução dos Spaces e da visibilidade `SPACE`.
- Interface: seletor de visibilidade na barra de ferramentas do editor (`web/src/components/MemoEditor/Toolbar/VisibilitySelector.tsx`).
- Necessidade de usuário: a [issue #126](https://github.com/usememos/memos/issues/126) pedia formas de compartilhar notas além de "privado" e "público".

### RF04 — Exportar e importar memos
O sistema deve permitir ao usuário exportar seus memos (texto e anexos) para um arquivo portável e importá-lo de volta.

**Evidências:**
- PR #6334 (`019ca26`, 16/09/2026).
- Interface: `web/src/components/Settings/MemoExportSection.tsx`, registrado nas configurações em `web/src/components/Settings/settingSections.ts`.
- Formato do arquivo: `docs/design/memos-export-format.md` (Markdown + manifesto JSON + anexos, em ZIP).
- Necessidade de usuário: a [discussão #2805](https://github.com/usememos/memos/discussions/2805) ("[FEATURE IDEA] export memos") registra o pedido da comunidade por exportação.

## 4.2 Requisitos não funcionais

### RNF01 — Segurança e controle de acesso
O sistema deve impedir que usuários não autorizados leiam memos privados ou restritos.

**Evidências:**
- Decisão de acesso centralizada em `CheckMemoReadContext` (`core/access/memo.go`). Para um memo `SPACE`, por exemplo: usuário anônimo recebe `Unauthenticated`; usuário autenticado que não é membro recebe `PermissionDenied` (a partir da linha 146).
- Conversão da decisão em erro da API: `server/api/v1/memo_access.go`, função `memoAccessDecisionError`.
- Testes automatizados das regras: `core/access/memo_test.go` (ex.: `TestCheckMemoReadSpaceAudience`).
- Documentação oficial: [usememos.com/docs/usage/sharing](https://usememos.com/docs/usage/sharing).

### RNF02 — Portabilidade da implantação
O sistema deve poder ser implantado em diferentes ambientes por meio de contêineres.

**Evidências:**
- Imagens Docker para `linux/amd64`, `linux/arm64` e `linux/arm/v7`, conforme a [documentação de implantação com Docker](https://usememos.com/docs/deploy/docker).
- No código: o workflow `.github/workflows/release.yml` (linhas 267–270) gera a imagem Docker de cada versão para essas três plataformas.

### RNF03 — Compatibilidade com diferentes bancos de dados
O sistema deve funcionar com SQLite, MySQL e PostgreSQL, à escolha de quem implanta.

**Evidências:**
- Um driver por banco: `store/db/sqlite`, `store/db/mysql` e `store/db/postgres`.
- Seleção via variáveis de ambiente `MEMOS_DRIVER` e `MEMOS_DSN` ([documentação](https://usememos.com/docs/deploy/docker#external-databases)).
- O workflow `backend-tests.yml` roda os testes da camada `store` nos três bancos.

### RNF04 — Manutenibilidade e qualidade verificadas automaticamente
Toda mudança deve passar por verificações automáticas de qualidade antes de ser integrada.

**Evidências:**
- `.github/workflows/backend-tests.yml`: `golangci-lint` (análise estática) e testes Go com detector de condição de corrida (`-race`) no pacote `server`.
- `.github/workflows/frontend-tests.yml`: `pnpm lint`, `pnpm test` e `pnpm build`.
- `.github/workflows/proto-linter.yml`: validação dos contratos Protocol Buffers.
- 187 arquivos de teste Go (`*_test.go`) na `v0.31.0`.

## 4.3 Histórias de usuário

### História 1 — Código nas notas
Como desenvolvedora de software, quero escrever blocos de código em Markdown nas minhas notas, para manter a sintaxe legível e colorida quando eu revisar minhas anotações.

**Cenário 1 — Destaque de sintaxe**
- **Dado que** estou no editor do Memos
- **Quando** escrevo um bloco de código com três crases e o nome da linguagem, e salvo o memo
- **Então** o memo exibido deve mostrar o código com destaque de sintaxe

**Cenário 2 — Edição do bloco**
- **Dado que** salvei um memo com um bloco de código
- **Quando** edito o conteúdo dentro do bloco e salvo
- **Então** a alteração deve ser salva e o bloco deve continuar formatado como código

### História 2 — Tags para encontrar resumos
Como estudante universitária, quero escrever a tag `#pesquisa` no texto da anotação, para conseguir filtrar e encontrar depois todos os resumos de artigos.

**Cenário 1 — Criação da tag**
- **Dado que** estou escrevendo um novo memo
- **Quando** incluo `#pesquisa` no texto e salvo
- **Então** a tag deve aparecer na lista de tags do menu lateral

**Cenário 2 — Filtro por tag**
- **Dado que** tenho vários memos, alguns com `#pesquisa`
- **Quando** clico na tag `#pesquisa` no menu lateral
- **Então** a linha do tempo deve mostrar apenas os memos que contêm essa tag

### História 3 — Compartilhar só com o Space
Como membro de uma equipe de projeto, quero restringir a visibilidade do meu memo ao meu Space, para compartilhar a informação com os colegas sem torná-la pública.

**Cenário 1 — Membro do Space consegue ler**
- **Dado que** pertenço a um Space e crio um memo com visibilidade `SPACE`
- **Quando** outro membro ativo do mesmo Space abre a linha do tempo
- **Então** ele deve conseguir ver o memo

**Cenário 2 — Quem está fora do Space não consegue ler**
- **Dado que** existe um memo com visibilidade `SPACE`
- **Quando** um usuário anônimo tenta abrir o link direto desse memo
- **Então** o sistema deve recusar o acesso com erro de não autenticado (`Unauthenticated`)
- **E quando** um usuário autenticado que não é membro do Space tenta abrir o mesmo link
- **Então** o sistema deve recusar com erro de permissão negada (`PermissionDenied`)

> Comportamento conferido em `core/access/memo.go` (caso `store.SpaceAudience`) e coberto por `TestCheckMemoReadSpaceAudience` em `core/access/memo_test.go`.

### História 4 — Cópia de segurança
Como usuária preocupada com meus dados, quero exportar todas as minhas anotações para um arquivo local, para ter uma cópia de segurança caso o servidor fique fora do ar.

**Cenário 1 — Exportação com sucesso**
- **Dado que** estou autenticada e abro a página de Configurações
- **Quando** clico na opção de exportar meus memos
- **Então** o sistema deve gerar e iniciar o download de um arquivo ZIP

**Cenário 2 — Conteúdo idêntico ao original**
- **Dado que** baixei o arquivo ZIP gerado pelo sistema
- **Quando** extraio o arquivo no meu computador
- **Então** o conteúdo de cada arquivo Markdown deve ser exatamente igual ao texto do memo correspondente

> O formato exige que o conteúdo seja gravado byte a byte, sem *front matter* nem reescrita de links: `docs/design/memos-export-format.md`, linha 190.