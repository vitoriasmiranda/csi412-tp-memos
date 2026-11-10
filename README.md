# Software Archaeology: investigando, compreendendo e evoluindo o Memos

Trabalho da disciplina **CSI412 – Engenharia de Software I** (UFOP / ICEA / DECSI), semestre 2026/2, prof. Igor Muzetti Pereira.

## Integrantes

- Vitória Miranda
- Vanderson Guimarães da Silva

## Software analisado

**Memos**: serviço open source e self-hosted para notas rápidas em formato de linha do tempo.

- Repositório original: https://github.com/usememos/memos
- Versão analisada: tag [`v0.31.0`](https://github.com/usememos/memos/tree/v0.31.0) (20/09/2026)
- Licença: MIT

## Objetivo

Investigar o Memos como se fôssemos entrar na equipe do projeto: entender o problema que ele resolve, seus usuários, requisitos, arquitetura e processo de desenvolvimento a partir de evidências do repositório, e então propor e implementar experimentalmente uma pequena evolução.

## Estrutura

| Pasta | Conteúdo |
|-------|----------|
| [`01-discovery`](01-discovery/) | O que é o software, usuários, personas e JTBD |
| [`02-processo`](02-processo/) | Evolução do projeto e relação com processos de software |
| [`03-requisitos`](03-requisitos/) | Requisitos funcionais e não funcionais, histórias de usuário e cenários |
| [`04-uml`](04-uml/) | Diagramas UML e correspondência com o código |
| [`05-evolucao`](05-evolucao/) | Proposta de evolução, MVP e análise de impacto |
| [`06-implementacao`](06-implementacao/) | Implementação experimental, revisão, testes e telas |
| [`07-etica`](07-etica/) | Questões éticas |
| [`08-ia`](08-ia/) | Registro do uso de IA (`ia.md`) |

## Proposta de evolução: adiar um memo até uma data

**Problema.** Nem toda anotação é útil agora. Hoje o usuário só pode deixar o memo na linha do tempo, onde ele distrai até a data em que importa, ou arquivá-lo, e aí ele não volta sozinho.

**Evidência.** A [issue #6388](https://github.com/usememos/memos/issues/6388) ("Due date & Hide-to-later option", aberta em 27/09/2026) pede exatamente isso, e outras quatro issues entre 2024 e 2026 pedem lembretes. A comunidade chegou a criar uma ferramenta externa ([memos-reminder](https://github.com/ertuil/memos-reminder)) para suprir a falta.

**Solução.** No editor, um botão com relógio permite escolher uma data e hora. Até essa data, o memo some da linha do tempo; depois, volta sozinho. Uma página **"Adiados"** lista os memos que ainda estão escondidos, para o usuário poder encontrá-los e cancelar o adiamento.

Detalhes, MVP e análise de impacto em [`05-evolucao/proposta.md`](05-evolucao/proposta.md); implementação, revisão, testes e telas em [`06-implementacao`](06-implementacao/).

## Como executar a intervenção

A implementação está no fork [vitoriasmiranda/memos](https://github.com/vitoriasmiranda/memos), branch `tp-base`.

Requisitos: Go 1.27, Node 24 e pnpm.

```bash
git clone https://github.com/vitoriasmiranda/memos
cd memos
git checkout tp-base

# servidor (porta 8081)
go run ./cmd/memos --port 8081

# em outro terminal: interface (abre em http://localhost:3001)
cd web
pnpm install
pnpm dev
```

No navegador, crie a conta de administrador, escreva um memo, clique no **relógio** ao lado da visibilidade, escolha uma data alguns minutos à frente e salve. O memo some da linha do tempo e aparece em **Avatar → Adiados**. Quando a data passar, ele volta para a linha do tempo.