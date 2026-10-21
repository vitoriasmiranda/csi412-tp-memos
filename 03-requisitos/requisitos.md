## 4.1 Requisitos Funcionais

*   **RF01: O sistema deve permitir a criação, edição e visualização de notas (memos) suportando formatação nativa em Markdown.**
    *   **Evidência:** No código-fonte, esta funcionalidade é sustentada pelo componente de interface construído com CodeMirror 6, visível no diretório `web/src/components/MemoEditor`. Adicionalmente, o *commit* `5a73d7d` comprova a reconstrução técnica deste editor para suportar fonte decorada sem perder o formato Markdown.

*   **RF02: O sistema deve permitir a organização, extração e filtragem de memos através do uso de tags (ex: `#arquitetura`) inseridas diretamente no corpo do texto.**
    *   **Evidência:** A implementação da busca e extração de tags está formalizada nos contratos da API definidos em Protocol Buffers (arquivo `api/v1/memo_service.proto`), que preveem parâmetros de filtragem por tag. Na interface, comprova-se pela renderização dinâmica do painel lateral de filtragem.

*   **RF03: O sistema deve permitir a definição de níveis granulares de visibilidade para os memos (Privado, Público ou restrito a um Space específico).**
    *   **Evidência:** O *Pull Request* `#6228` comprova a implementação no *backend* do suporte a múltiplos *Spaces* e regras de autorização de visibilidade (`SPACE`, `PRIVATE`, `PUBLIC`). Na aplicação em execução, a evidência é o seletor dropdown de visibilidade na caixa de criação da interface.

*   **RF04: O sistema deve permitir ao utilizador exportar e importar a sua base de conhecimento (textos e anexos) de forma portável.**
    *   **Evidência:** O *Pull Request* `#6334` documenta a implementação exata desta funcionalidade, descrevendo a lógica de empacotamento das anotações em formato Markdown e manifesto JSON num arquivo ZIP. Na interface, a funcionalidade é acedida no painel de "Configurações" (Settings).