# 1. O que é o Memos?

O Memos é um serviço open source e self-hosted para registrar notas rápidas, ideias e conhecimento. A interface funciona como uma linha do tempo (timeline), parecida com um microblog privado: o usuário escreve e salva sem precisar criar pastas ou estruturar documentos longos.

> **Versão analisada:** tag [`v0.31.0`](https://github.com/usememos/memos/releases/tag/v0.31.0), publicada em 20/09/2026.
> Depois dela o projeto adotou versionamento por calendário (`26.10.0`, `26.11.0`; PR #6362). Todas as evidências e diagramas deste trabalho se referem à `v0.31.0`.

## 1.1 Análise geral do sistema

- **Problema que resolve:** a dificuldade de capturar informações, ideias soltas ou trechos de código rapidamente, num lugar centralizado, privado e pesquisável, evitando que as anotações fiquem espalhadas em várias ferramentas e arquivos. A linha do tempo contínua elimina a dúvida inicial de "onde guardo isto?".
- **Público-alvo:** pessoas e equipes que querem uma ferramenta leve e privada para registros diários (journaling), gestão de conhecimento pessoal (PKM) e organização de material de estudo ou trabalho.
- **Principais usuários:** quem lida com muitas informações fragmentadas no dia a dia, como estudantes, desenvolvedores e profissionais de TI. Como o Memos é self-hosted, o usuário inicial costuma ter perfil técnico (precisa instalar via Docker ou binário).
  > **[A FAZER]** Sustentar com evidência real: citar 2–3 issues ou discussões do GitHub em que usuários descrevem como usam o Memos (links).
- **Principais funcionalidades:**
  - Escrita em Markdown, com editor baseado em CodeMirror (`web/src/components/MemoEditor/`).
  - Organização por tags escritas no próprio texto (ex.: `#estudo`), extraídas automaticamente (`markdown/extensions/tag.go`).
  - Filtros e busca (filtro `tags` em `proto/api/v1/memo_service.proto`).
  - Níveis de visibilidade: `PRIVATE`, `PROTECTED`, `PUBLIC` e `SPACE`.
  - Anexos (imagens e arquivos), com armazenamento local ou compatível com S3.
  - Exportação e importação de memos em ZIP (PR #6334).
  - API definida em Protocol Buffers, exposta via Connect RPC e REST (gRPC-Gateway).
- **Tecnologias utilizadas:** Go (backend), React + TypeScript (frontend), Protocol Buffers (contratos da API), SQLite como banco padrão, com suporte também a MySQL e PostgreSQL (`store/db/`). Distribuído como imagem Docker.
- **Licença:** MIT (arquivo `LICENSE`).
- **Versão analisada:** `v0.31.0`.

## 1.2 Personas e Jobs to Be Done (JTBD)

### Persona 1: Estudante de Engenharia de Software
- **Perfil:** estudante universitário que cursa várias disciplinas ao mesmo tempo. Trabalha com trechos de código (ex.: PHP ou C) e precisa anotar explicações de erros e soluções encontradas durante os projetos práticos.
- **JTBD:** "Quando estou depurando meu código e encontro a solução de um erro, quero salvar esse trecho rapidamente numa nota com a tag da linguagem, para poder pesquisar e reutilizar essa solução depois sem perder o ritmo de trabalho."
- **Evidência:** suporte a blocos de código com destaque de sintaxe (`web/src/components/MemoContent/highlight.ts`, usando `highlight.js`).
  > **[A FAZER]** Link de uma issue real sobre blocos de código / destaque de sintaxe.

### Persona 2: Profissional de suporte técnico de TI
- **Perfil:** trabalha numa empresa atendendo chamados de clientes. Precisa documentar procedimentos repetitivos e comandos úteis para não esquecê-los.
- **JTBD:** "Quando descubro uma nova forma de resolver um chamado frequente, quero registrar esse procedimento na minha linha do tempo com tags de classificação, para construir minha própria base de conhecimento sem depender de sistemas complexos."
- **Evidência:** tags extraídas do texto e filtro por tag; visibilidade `SPACE` permite compartilhar com a equipe.
  > **[A FAZER]** Link de uma issue real sobre uso como base de conhecimento / compartilhamento com equipe.