## 4.1 Requisitos Funcionais

*   **RF01: O sistema deve permitir a criação, edição e visualização de notas (memos) suportando formatação nativa em Markdown.**
    *   **Evidência:** No código-fonte, esta funcionalidade é sustentada pelo componente de interface construído com CodeMirror 6, visível no diretório `web/src/components/MemoEditor`. Adicionalmente, o *commit* `5a73d7d` comprova a reconstrução técnica deste editor para suportar fonte decorada sem perder o formato Markdown.

*   **RF02: O sistema deve permitir a organização, extração e filtragem de memos através do uso de tags (ex: `#arquitetura`) inseridas diretamente no corpo do texto.**
    *   **Evidência:** A implementação da busca e extração de tags está formalizada nos contratos da API definidos em Protocol Buffers (arquivo `api/v1/memo_service.proto`), que preveem parâmetros de filtragem por tag. Na interface, comprova-se pela renderização dinâmica do painel lateral de filtragem.

*   **RF03: O sistema deve permitir a definição de níveis granulares de visibilidade para os memos (Privado, Público ou restrito a um Space específico).**
    *   **Evidência:** O *Pull Request* `#6228` comprova a implementação no *backend* do suporte a múltiplos *Spaces* e regras de autorização de visibilidade (`SPACE`, `PRIVATE`, `PUBLIC`). Na aplicação em execução, a evidência é o seletor dropdown de visibilidade na caixa de criação da interface.

*   **RF04: O sistema deve permitir ao utilizador exportar e importar a sua base de conhecimento (textos e anexos) de forma portável.**
    *   **Evidência:** O *Pull Request* `#6334` documenta a implementação exata desta funcionalidade, descrevendo a lógica de empacotamento das anotações em formato Markdown e manifesto JSON num arquivo ZIP. Na interface, a funcionalidade é acedida no painel de "Configurações" (Settings).

## 4.2 Requisitos Não Funcionais

* **RNF01: O sistema deve garantir a segurança e o controle de acesso às anotações, respeitando as permissões definidas e impedindo que usuários não autorizados visualizem ou modifiquem informações privadas.**
  * **Evidência:** O Memos possui mecanismos de autenticação e diferentes níveis de visibilidade: `PRIVATE` (anotações privadas), `PROTECTED` (acessíveis a usuários autenticados), `PUBLIC` (anotações públicas) e `SPACE` (acessíveis aos membros autorizados de um espaço). Essas regras estão descritas na [documentação oficial de compartilhamento](https://usememos.com/docs/usage/sharing) e demonstram a existência de mecanismos de proteção das informações.

* **RNF02: O sistema deve permitir sua implantação e execução em diferentes ambientes computacionais, garantindo portabilidade por meio de tecnologias de conteinerização.**
  * **Evidência:** A [documentação oficial de implantação com Docker](https://usememos.com/docs/deploy/docker) apresenta imagens compatíveis com diferentes arquiteturas, incluindo `linux/amd64`, `linux/arm64` e `linux/arm/v7`. Essa característica permite executar o Memos em diferentes ambientes sem a necessidade de instalar manualmente todas as dependências da aplicação.

* **RNF03: O sistema deve oferecer compatibilidade com diferentes sistemas gerenciadores de bancos de dados, permitindo flexibilidade na escolha da solução de armazenamento conforme o ambiente de implantação.**
  * **Evidência:** A [documentação oficial do Memos](https://usememos.com/docs/deploy/docker#external-databases) apresenta suporte aos bancos de dados SQLite, MySQL e PostgreSQL. A configuração é realizada por meio das variáveis de ambiente `MEMOS_DRIVER` e `MEMOS_DSN`, permitindo selecionar o mecanismo de persistência utilizado pela aplicação.

* **RNF04: O sistema deve favorecer a manutenibilidade e a qualidade do código por meio de testes automatizados, análise estática e verificações contínuas durante o desenvolvimento.**
  * **Evidência:** O repositório oficial utiliza **GitHub Actions** para executar verificações automatizadas, incluindo testes do backend e frontend, análise estática, verificação de formatação, validação de tipos e build da aplicação. Essas práticas podem ser observadas nos [workflows do GitHub Actions](https://github.com/usememos/memos/actions) e na [documentação oficial de testes](https://usememos.com/docs/development/testing), contribuindo para identificar defeitos e reduzir riscos durante a evolução do software.

## Histórias de Usuário

**História 1**
Como desenvolvedora de software, quero utilizar blocos de código formatados em Markdown nas minhas notas, para manter a sintaxe legível e colorida quando for rever as minhas anotações.
*   **Cenário de Teste 1 (Renderização de Código):**
    *   **Dado que** estou no editor principal do Memos.
    *   **Quando** digito um bloco de código usando crases triplas e a tag da linguagem.
    *   **Então** o sistema deve aplicar o destaque de sintaxe corretamente no texto visualizado.
*   **Cenário de Teste 2 (Edição de Bloco):**
    *   **Dado que** guardei um memo com um bloco de código.
    *   **Quando** clico para editar e altero o conteúdo dentro do bloco.
    *   **Então** o sistema deve guardar a alteração sem quebrar a formatação original.

**História 2**
Como estudante universitária, quero adicionar a tag `#pesquisa` diretamente no texto da minha anotação, para conseguir filtrar e encontrar todos os resumos de artigos posteriormente.
*   **Cenário de Teste 1 (Criação de Tag):**
    *   **Dado que** estou a redigir um novo memo.
    *   **Quando** incluo a palavra `#pesquisa` no texto e guardo.
    *   **Então** a tag deve ser extraída e listada na secção "Tags" do menu lateral.
*   **Cenário de Teste 2 (Filtro por Tag):**
    *   **Dado que** tenho vários memos na minha linha do tempo.
    *   **Quando** clico na tag `#pesquisa` no menu lateral.
    *   **Então** a linha do tempo deve ser atualizada para exibir exclusivamente os memos que contêm essa marcação.

**História 3**
Como membro de uma equipa de projeto, quero definir a visibilidade do meu memo apenas para o meu "Space", para partilhar a informação com os meus colegas de trabalho sem a tornar pública na internet.
*   **Cenário de Teste 1 (Publicação no Space):**
    *   **Dado que** pertenço a um "Space" configurado no servidor.
    *   **Quando** crio um memo e seleciono a visibilidade `SPACE`.
    *   **Então** apenas os utilizadores pertencentes a esse Space devem conseguir visualizar a anotação na sua linha do tempo.
*   **Cenário de Teste 2 (Isolamento de Dados):**
    *   **Dado que** um memo foi definido com a visibilidade `SPACE`.
    *   **Quando** um utilizador anónimo ou de outro Space tenta aceder ao link direto desse memo.
    *   **Então** o sistema deve bloquear o acesso e apresentar uma mensagem de erro de autorização.

**História 4**
Como utilizadora preocupada com os meus dados, quero exportar todas as minhas anotações para um ficheiro local, para ter uma cópia de segurança caso o servidor fique offline.
*   **Cenário de Teste 1 (Exportação com Sucesso):**
    *   **Dado que** estou autenticada e acedo à página de "Configurações".
    *   **Quando** clico na opção de exportar os meus dados.
    *   **Então** o sistema deve processar o pedido e iniciar a transferência de um ficheiro ZIP.
*   **Cenário de Teste 2 (Validação do Conteúdo):**
    *   **Dado que** transferi o ficheiro ZIP gerado pelo sistema.
    *   **Quando** extraio o conteúdo da pasta no meu computador.
    *   **Então** o conteúdo dos ficheiros Markdown deve corresponder exatamente ao texto das anotações que criei na interface.