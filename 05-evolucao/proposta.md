# 6. Evolução proposta: adiar um memo até uma data

> Análise feita sobre a tag `v0.31.0` do [usememos/memos](https://github.com/usememos/memos/tree/v0.31.0).

## 6.1 Proposta

### Problema

O Memos é feito para registrar ideias rapidamente, mas nem toda ideia é útil **agora**. Uma tarefa para o mês que vem, um lembrete de renovar um documento ou uma ideia para retomar depois das provas entram na linha do tempo junto com tudo o que é atual.

Hoje o usuário tem duas saídas, e as duas são ruins:

- **Deixar o memo na linha do tempo:** ele ocupa espaço e distrai até a data em que passa a importar. Se o usuário cria muitos memos, ele acaba enterrado e esquecido justamente quando deveria reaparecer.
- **Arquivar o memo** (estado `ARCHIVED`, página `web/src/pages/Archived.tsx`): ele some da linha do tempo, mas **não volta sozinho**. O usuário precisa lembrar de abrir a página de arquivados e desarquivar na data certa, o que anula o objetivo.

### Usuário afetado

Quem usa o Memos para tarefas e ideias com data, como as duas personas da seção 1:

- a estudante que anota algo para retomar no fim do semestre;
- o profissional de suporte que precisa revisar um procedimento numa data futura.

### Evidências

| Evidência | O que mostra |
|-----------|--------------|
| [Issue #6388](https://github.com/usememos/memos/issues/6388) — "Due date & Hide-to-later option" (aberta em 27/09/2026, sem resposta) | Pedido exatamente desta funcionalidade: guardar ideias sem vê-las na linha do tempo até que sejam relevantes, com um ícone de relógio no editor para escolher a data. |
| [Issue #2797](https://github.com/usememos/memos/issues/2797) — "Setting reminders for memos" | Usuário que usa o Memos para notas soltas e precisa agir sobre tarefas até um prazo. |
| [Issue #3599](https://github.com/usememos/memos/issues/3599) — "Basic Memo Reminders" | Pedido de um lembrete simples por data e hora. |
| [Issue #4007](https://github.com/usememos/memos/issues/4007) — "Remind me about a task at a specific time" | Pedido de lembrete em data e hora; na discussão, é citado que o tema ainda não teve tração. |
| [Issue #4583](https://github.com/usememos/memos/issues/4583) — "Reminders" | Usuário vindo do Google Keep; outro usuário comenta que continua no Keep por falta desse recurso. |
| Projeto externo [ertuil/memos-reminder](https://github.com/ertuil/memos-reminder) | A comunidade criou uma ferramenta separada (via webhook + e-mail) para suprir a falta de lembretes, o que confirma a necessidade. |
| Código da `v0.31.0` | O `Memo` não tem nenhum campo de data futura: só `create_time` e `update_time` (`proto/api/v1/memo_service.proto`, linhas 225 e 229). |

O pedido aparece em issues de 2024 a 2026, o que mostra que é uma necessidade **recorrente** e ainda **não atendida**.

### Solução proposta

Permitir que o autor de um memo defina uma data e hora até a qual o memo fica **oculto da linha do tempo**. Ao chegar a data, o memo **volta sozinho** para a listagem normal, como um lembrete.

- No editor, um botão com ícone de relógio, ao lado do seletor de visibilidade, abre um seletor de data e hora.
- O memo continua existindo normalmente: pode ser aberto pelo link direto e editado.
- A volta é automática porque o sistema compara a data de adiamento com o horário atual **no momento da consulta**. Não é preciso nenhuma tarefa agendada no servidor.

### Benefício esperado

- Linha do tempo mais limpa, mostrando só o que importa no momento.
- Ideias e tarefas futuras não se perdem: reaparecem na hora certa.
- Atende a um pedido recorrente da comunidade sem depender de ferramentas externas.

### Prioridade

**Média.**

- **A favor de priorizar:** é um pedido recorrente (5 issues entre 2024 e 2026, uma delas aberta agora), a comunidade já criou uma solução externa, e a mudança é pequena e não exige alteração no esquema do banco (ver 6.3).
- **Contra:** não corrige falha nem segurança, e há um contorno manual (arquivar e desarquivar). Nenhum mantenedor respondeu à issue #6388 ainda, então a proposta precisaria ser discutida com eles antes de virar um PR no projeto original.

---

## 6.2 MVP (Produto Mínimo Viável)

### O que será desenvolvido

1. **Campo de adiamento no memo:** uma data/hora opcional `hide_until`, definida pelo autor.
2. **Filtro na listagem:** memos com `hide_until` no futuro não aparecem na linha do tempo; quando a data passa, voltam a aparecer.
3. **Controle no editor:** botão para escolher a data de adiamento e para remover o adiamento.
4. **Testes automatizados** do filtro: memo adiado não aparece; memo com data já vencida aparece; memo sem adiamento aparece.

### Valor entregue

O usuário consegue registrar algo agora e só voltar a vê-lo na data escolhida, que é o núcleo do pedido da issue #6388.

### Fora do MVP

| Fica de fora | Por quê |
|--------------|---------|
| Notificação (e-mail, push, inbox) quando o memo reaparece | Exige agendamento no servidor e integração com SMTP/notificações; é a parte mais cara e não é necessária para o memo voltar à linha do tempo. |
| Data de vencimento (*due date*) separada da data de adiamento | A issue #6388 pede as duas, mas o adiamento sozinho já resolve o problema principal. |
| Atalhos de antecedência ("1 dia antes", "1 semana antes") | Melhoria de usabilidade que depende da data de vencimento. |
| Adiamentos recorrentes (toda semana, todo mês) | Aumenta muito a complexidade das regras. |
| Página para ver só os memos adiados | Útil, mas o usuário ainda consegue achar o memo pela busca ou pelo link. |

### Justificativa das escolhas

- O MVP ataca o problema central ("não quero ver isto agora, mas não quero esquecer") com o menor número de partes alteradas.
- Fazer a volta do memo **na hora da consulta**, em vez de com uma tarefa agendada, evita criar um processo em segundo plano e elimina o risco de o memo "não voltar" se o servidor estiver desligado na data.
- Guardar a data dentro do `payload` do memo, que é um campo JSON já existente, evita migrações nos três bancos suportados (SQLite, MySQL e PostgreSQL).
- Notificações ficaram de fora porque já existe uma alternativa externa (memos-reminder) e porque elas dependeriam de configuração de SMTP pelo administrador.

---

## 6.3 Análise de impacto

| Tipo | Parte afetada | Mudança | Impacto |
|------|---------------|---------|---------|
| **API (contrato)** | `proto/api/v1/memo_service.proto`, mensagem `Memo` | Novo campo opcional `hide_until` (Timestamp) | Médio. Campo novo e opcional: clientes antigos continuam funcionando (mudança compatível). |
| **API (código gerado)** | `proto/gen/api/v1/`, `proto/gen/openapi.yaml`, `web/src/types/proto/api/v1/` | Regenerar a partir do `.proto` | Baixo. Gerado automaticamente pelo `buf`. |
| **Modelo de dados** | `proto/store/memo.proto`, mensagem `MemoPayload` | Novo campo `hide_until` dentro do payload | Baixo. O payload é salvo como JSON. |
| **Banco de dados** | `store/db/sqlite`, `store/db/mysql`, `store/db/postgres` | **Nenhuma mudança de esquema** | Baixo. O campo `payload` já é JSON, então não há migração. |
| **Serviço da API** | `server/api/v1/memo_service.go` (`CreateMemo`, `UpdateMemo`) | Aceitar `hide_until` na criação e o caminho `hide_until` no `update_mask` da edição | Médio. Só o autor pode definir o adiamento. |
| **Conversão** | `server/api/v1/memo_service_converter.go` | Converter o campo entre a API e o payload | Baixo. |
| **Filtro de listagem** | `filter/schema.go` e `filter/render.go` | Novo campo filtrável `hide_until`, lido do JSON do payload, como já é feito com `has_task_list` (`filter/schema.go`, linha 207) | Alto. É o ponto central; precisa gerar SQL correto nos três bancos. |
| **Listagem de memos** | `server/api/v1/memo_service.go` (`ListMemos`) | Excluir memos com `hide_until` no futuro da listagem padrão | Alto. O filtro deve ser aplicado **no banco**, antes da paginação, senão as páginas ficam com menos itens que o esperado. |
| **Controle de acesso** | `core/access/memo.go` | Nenhuma mudança | Nenhum. Adiar não muda quem pode ler o memo. |
| **Interface: editor** | `web/src/components/MemoEditor/Toolbar/` (novo componente ao lado de `VisibilitySelector.tsx`) | Botão com seletor de data | Médio. |
| **Interface: exibição** | `web/src/components/MemoView/` | Indicar que um memo está adiado (quando aberto pelo link) | Baixo. |
| **MCP** | `server/mcp/` | Nenhuma mudança manual | Baixo. As ferramentas MCP são geradas do `openapi.yaml` (PR #6026), então o novo campo é exposto automaticamente. |
| **Exportação** | `core/memoexport/`, `docs/design/memos-export-format.md` | Decidir se `hide_until` entra no manifesto do ZIP | Médio. Se ficar de fora, um memo adiado volta visível ao ser importado. |
| **Testes** | `filter/engine_test.go`, `server/test/`, `store/test/` | Testes do novo filtro e do comportamento da listagem nos três bancos | Médio. |
| **Documentação** | `docs/design/` e documentação de uso | Documento de design do adiamento, seguindo o padrão do projeto | Baixo. |

### Riscos

- **Fuso horário:** a data é escolhida no fuso do usuário e comparada no servidor. Ela deve ser enviada e guardada em UTC (o projeto já usa Timestamp em UTC na API).
- **Paginação:** filtrar depois de buscar no banco quebraria a contagem das páginas. O código atual já evita isso no controle de acesso (comentário de `newMemoAccessScope`, em `server/api/v1/memo_access.go`), e o adiamento deve seguir o mesmo princípio.
- **Exportação/importação:** sem incluir o campo no arquivo, o adiamento se perde numa migração de instância.