# Análise da Evolução e do Processo de Desenvolvimento do Memos

## 1. Histórico e Evolução do Projeto

O Memos apresenta uma evolução incremental, com mudanças registradas publicamente por meio de commits, issues, pull requests e releases.

Ao analisar o histórico do projeto, destacam-se quatro acontecimentos importantes que demonstram sua evolução técnica e funcional.

### 1.1. Criação do Produto Mínimo — Dezembro de 2021

O primeiro commit do projeto, realizado em 8 de dezembro de 2021, já apresentava elementos fundamentais, como:

- Gerenciamento de usuários;
- Autenticação;
- Criação e gerenciamento de memos;
- Persistência de dados utilizando SQLite.

Poucos dias depois, o projeto recebeu uma **interface web** e um **Dockerfile**, permitindo sua utilização e implantação de maneira mais simples.

---

### 1.2. Remoção da API Legada — Abril e Maio de 2024

Em abril de 2024, foi removida a API antiga do projeto.

Posteriormente, a versão baseada em Protocol Buffers, Connect RPC e gRPC-Gateway foi consolidada como a API principal. Essa mudança foi publicada na release `v0.22.0`, em maio de 2024.

#### Principais consequências

- Redução da duplicação entre APIs;
- Centralização dos contratos em Protocol Buffers;
- Facilidade na geração de clientes;
- Simplificação da arquitetura interna;
- Necessidade de adaptação das integrações que utilizavam a API antiga.

---

### 1.3. Reconstrução do Editor e Integração MCP — Junho e Julho de 2026

O commit `5a73d7d`, realizado em 28 de junho de 2026, reconstruiu o editor utilizando CodeMirror 6.

O novo editor passou a trabalhar com uma abordagem de *fonte decorada*, na qual o Markdown continua visível, enquanto títulos, tags, links e menções recebem comportamentos e estilos durante a edição.

Além disso, a release `v0.30.0`, publicada em julho de 2026, introduziu uma integração **MCP**, derivada da especificação OpenAPI do projeto.

#### Principais contribuições

- **Evolução perfectiva:** melhoria da experiência de escrita;
- **Reuso de software:** aproveitamento da API existente;
- **Integração:** ampliação da capacidade de integração programática;
- **Modernização:** substituição de um subsistema central sem abandonar o formato Markdown.

---

### 1.4. Spaces, Colaboração e Portabilidade — Agosto e Setembro de 2026

O pull request `#6228` introduziu o suporte a múltiplos Spaces.

#### Funcionalidades adicionadas

- Criação de Spaces;
- Participação de diferentes membros;
- Definição de papéis de administrador e usuário;
- Visibilidade `SPACE`;
- Regras de autorização específicas;
- Migrações para SQLite, MySQL e PostgreSQL.

Posteriormente, o pull request `#6334` implementou a funcionalidade de **exportação e importação de memos por meio de arquivos ZIP**.

O formato de exportação inclui:

- Conteúdo em Markdown;
- Manifesto JSON;
- Anexos associados aos memos.

---

## 2. Processo de Desenvolvimento e Práticas de Engenharia de Software

O **Memos** é desenvolvido publicamente no GitHub, seguindo um fluxo incremental baseado em:

1. Issues;
2. Branches;
3. Commits;
4. Pull requests;
5. Revisão de código;
6. Integração contínua.


### 2.1. Issues Estruturadas

O projeto possui modelos diferentes para o registro de **defeitos** e **solicitações de funcionalidades**.

Os formulários solicitam informações como:

- Problema observado;
- Comportamento esperado;
- Passos para reprodução;
- Versão utilizada;
- Motivação da solicitação.

Essa organização melhora a compreensão das necessidades antes da implementação.

### 2.2. Pull Requests Revisáveis

As mudanças são apresentadas por meio de **pull requests**, com escopo identificável.

Funcionalidades maiores, como MCP, Spaces e exportação de dados, foram discutidas e revisadas antes da integração.

Essa prática proporciona **rastreabilidade** entre o problema identificado, a implementação, os testes e a revisão do código.

### 2.3. Integração Contínua

O projeto utiliza **GitHub Actions** para automatizar diferentes etapas de verificação da qualidade do software.

Os workflows executam:

- Testes do backend e do frontend;
- Análise estática de código;
- Verificação de formatação;
- Verificação de tipos;
- Build da aplicação;
- Validação de Protocol Buffers;
- Testes de migração;
- Verificações com diferentes bancos de dados.

Essas verificações fornecem feedback automatizado antes da integração das alterações.

### 2.4. Commits Convencionais

As mensagens de commit utilizam prefixos padronizados, como:

| Prefixo | Finalidade |
|---------|------------|
| `feat` | Adição de funcionalidades |
| `fix` | Correção de defeitos |
| `refactor` | Refatoração de código |
| `test` | Adição ou alteração de testes |
| `docs` | Atualização da documentação |
| `chore` | Tarefas de manutenção |

Essa padronização facilita a leitura do histórico e contribui para a geração automática das notas de versão.

### 2.5. Releases Automatizadas

O projeto utiliza **Release Please** para automatizar parte do processo de lançamento de novas versões.

A ferramenta auxilia em:

- Preparação de versões;
- Atualização de changelogs;
- Organização de funcionalidades;
- Registro de correções;
- Documentação de melhorias de desempenho.

### 2.6. Decisões Arquiteturais Documentadas

O repositório contém **ADRs (Architecture Decision Records)** e documentos de design que registram decisões relacionadas à arquitetura do sistema.

Esses documentos apresentam:

- Contexto da decisão;
- Objetivos;
- Alternativas consideradas;
- Decisões adotadas;
- Consequências;
- Elementos fora do escopo.

Essa documentação facilita a compreensão das escolhas técnicas ao longo do desenvolvimento.

### 2.7. Testes de Migração

Como o Memos oferece suporte a **SQLite, MySQL e PostgreSQL**, mudanças relacionadas à persistência de dados são acompanhadas por migrações e verificações específicas.

Essa prática contribui para reduzir riscos de perda de informações, inconsistências e incompatibilidades entre versões.

---

## 3. Relação com os Conceitos de Processos de Software

As práticas observadas no desenvolvimento do Memos apresentam uma forte relação com conceitos estudados em Engenharia de Software, especialmente:

- Desenvolvimento incremental;
- Extreme Programming (XP);
- Reuso de software.

### 3.1. Desenvolvimento Incremental

O Memos não foi construído como um produto completo em uma única etapa.

Seu histórico apresenta entregas sucessivas, como:

1. Autenticação e persistência com SQLite;
2. Interface web e suporte a Docker;
3. Modernização da API;
4. Reconstrução do editor;
5. Integração MCP;
6. Implementação de Spaces;
7. Exportação e importação de dados.

Cada entrega acrescenta funcionalidades ou melhorias ao sistema, agregando valor real e utilizável ao produto.


### 3.2. Extreme Programming (XP)

O projeto apresenta práticas características da metodologia **XP (Extreme Programming)**, entre elas:

- **Integração contínua:** alterações são verificadas frequentemente;
- **Testes automatizados:** contribuem para identificar problemas e preservar a qualidade;
- **Refatoração:** permite melhorar a estrutura do código;
- **Mudanças pequenas:** facilitam a revisão e a integração;
- **Feedback rápido:** possibilita identificar e corrigir problemas durante o desenvolvimento.

Essas práticas contribuem para a melhoria contínua do código, a qualidade do software e a adaptação rápida às mudanças.

Dessa forma, o projeto apresenta características importantes de XP, aplicando princípios que tornam o desenvolvimento mais ágil, eficiente e colaborativo.

### 3.3. Reuso de Software

O Memos utiliza diferentes tecnologias, bibliotecas e componentes existentes para construir suas funcionalidades.

#### Tecnologias reutilizadas

| Tecnologia | Aplicação |
|------------|-----------|
| React | Desenvolvimento da interface |
| CodeMirror | Implementação do editor |
| Echo | Desenvolvimento do backend |
| Connect RPC | Comunicação entre serviços |
| Protocol Buffers | Definição de contratos |
| Drivers de banco de dados | Integração com bancos de dados |
| SDK compatível com S3 | Integração com armazenamento de objetos |
| OpenAPI | Especificação e integração de APIs |
| Componentes internos | Reaproveitamento de funcionalidades |

O reuso dessas tecnologias reduz o esforço necessário para implementar funcionalidades já disponíveis em soluções consolidadas.

