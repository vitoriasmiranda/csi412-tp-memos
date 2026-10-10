# 1. O que é o Memos?
O Memos é um serviço open source e self-hosted focado na captura de notas rápidas, pensamentos e registos de conhecimento. O sistema apresenta uma interface em estilo "linha do tempo" (timeline), semelhante a um microblog privado, permitindo aos utilizadores guardar informações sem a necessidade de criar pastas complexas ou estruturar documentos extensos.

## 1.1 Análise Geral do Sistema
- **Problema que resolve:** A dificuldade de capturar informações, ideias soltas ou trechos de código rapidamente num local centralizado, privado e pesquisável, evitando a fragmentação de anotações em várias ferramentas ou ficheiros dispersos. O Memos elimina a fricção inicial de "onde guardar isto?" ao focar-se numa linha do tempo contínua.
- **Público-alvo:** Indivíduos e equipes que procuram uma ferramenta leve e privada para registos diários (journaling), gestão de conhecimento pessoal (PKM) e organização de material de estudo ou trabalho.
- **Principais utilizadores:** Estudantes universitários, engenheiros de software, investigadores, escritores e profissionais que lidam com um grande fluxo de informações fragmentadas.
- **Principais funcionalidades:** 
  * Suporte para formatação em Markdown. 
  * Organização ágil através de tags (ex: #estudo, #codigo). 
  * Filtros avançados e pesquisa integrada. 
  * Possibilidade de partilha pública de notas específicas. 
  * Suporte para anexos (imagens e ficheiros). 
  * Integração via API (REST).
- **Tecnologias utilizadas:** Go (backend), React / TypeScript (frontend) e SQLite (banco de dados). Utiliza Docker para facilitar a implementação.
- **Licença:** Licença MIT.
- **Versão analisada:** v0.31.0

## 1.2 Personas e Jobs to Be Done (JTBD)
Para compreender melhor o impacto e o uso do Memos, definimos duas personas principais:

### Persona 1: O Estudante de Engenharia de Software
- **Perfil:** Estudante universitário (como nós) que está a aprender várias disciplinas complexas ao mesmo tempo. Ele lida frequentemente com blocos de código (ex: PHP ou C) e precisa de anotar explicações sobre erros ou soluções de arquitetura que encontra durante o desenvolvimento de projetos práticos.
- **Job to Be Done (JTBD):** "Quando estou a depurar o meu código e encontro a solução para um erro, quero guardar esse fragmento de código rapidamente numa nota com a tag da linguagem, para que eu possa pesquisar e reutilizar essa lógica futuramente sem perder o ritmo de trabalho."

### Persona 2: O Profissional de Suporte Técnico de TI
- **Perfil:** Trabalho em uma empresa resolvendo chamados de clientes. Durante o dia, ele precisa de documentar procedimentos repetitivos e comandos úteis para não os esquecer.
- **Job to Be Done (JTBD):** "Quando descubro uma nova forma de resolver um chamado frequente dos clientes, quero registar esse procedimento na minha linha do tempo com tags de classificação, para construir a minha própria base de conhecimento sem depender de sistemas complexos."