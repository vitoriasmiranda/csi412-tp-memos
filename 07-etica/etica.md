# 8. Ética e responsabilidade

> Evidências conferidas na tag `v0.31.0` do [usememos/memos](https://github.com/usememos/memos/tree/v0.31.0).

## 8.1 Privacidade: o administrador lê tudo, e "adiar" não é "esconder"

### O problema

O Memos é *self-hosted*: quem instala o servidor é o administrador da instância, e ele tem acesso a tudo. No código, o administrador passa por todas as verificações de acesso ao abrir um memo pelo nome, inclusive os marcados como `PRIVATE` (`core/access/memo.go`, função `CheckMemoReadContext`, linhas 107–110: *"A structurally valid memo is readable by name to an instance administrator regardless of audience"*). Para quem usa o Memos numa instância de outra pessoa (da empresa, de um amigo, de um grupo de estudo), "privado" quer dizer "privado para os outros usuários", mas **não** para quem administra o servidor. A interface não deixa isso claro.

A nossa funcionalidade traz uma questão parecida. Um memo adiado some da linha do tempo, mas **continua com a mesma visibilidade**: se ele for `PUBLIC`, qualquer pessoa com o link ainda consegue abri-lo. Adiar é uma forma de organizar, não de proteger. Um usuário pode confundir as duas coisas e achar que "escondeu" uma informação sensível.

### A responsabilidade do engenheiro

- **Ser transparente sobre quem pode ver os dados.** O sistema deveria informar, por exemplo na página de configurações ou no seletor de visibilidade, que o administrador da instância tem acesso a todos os memos.
- **Não deixar uma funcionalidade parecer o que ela não é.** Por isso, na nossa implementação, o texto do seletor diz que o memo "sai da linha do tempo e volta a aparecer nesta data", sem usar palavras como "ocultar" ou "esconder" no sentido de privacidade.
- **Aplicar o mínimo de acesso necessário.** A página "Adiados" que implementamos lista **só os memos do próprio usuário** (`memoFind.CreatorID = &currentUser.ID` em `ListMemos`), e o teste `TestListMemosHiddenOnly` confirma que outro usuário não vê nada. Sem essa restrição, a página poderia revelar a existência de memos adiados de outras pessoas.

## 8.2 Responsabilidade sobre vulnerabilidades: SSRF e a política de segurança

### O problema

O Memos faz requisições para endereços externos em pelo menos dois pontos: ao gerar a prévia de links citados nos memos (`internal/linkmeta/html_meta.go`) e ao enviar webhooks (`internal/webhook/webhook.go`). Isso abre espaço para **SSRF** (*Server-Side Request Forgery*): um usuário escreve um link apontando para um endereço interno da rede onde o servidor está instalado, e o servidor faz a requisição por ele, expondo serviços que deveriam estar protegidos.

O histórico mostra que o problema foi tratado várias vezes ao longo de 2026:

| Data | Commit | O que mudou |
|------|--------|-------------|
| 23/02/2026 | `150371d` | Corrige uma vulnerabilidade de SSRF no envio de webhooks |
| 07/03/2026 | `cd5816c` (PR #5694) | Adiciona a opção `--allow-private-webhooks`, que permite desligar a proteção |
| 07/09/2026 | `607dc28` (PR #6284) | Bloqueia mais um tipo de endereço que escapava da proteção |
| 19/09/2026 | `f4d97fa` | Bloqueia a faixa CGNAT (`100.64.0.0/10`), que a verificação padrão do Go não considera privada |

Hoje a prévia de links recusa endereços internos (`ErrInternalIP`, linha 20; verificação de *loopback*, privados e *link-local* na linha 116).

Ao mesmo tempo, a política de segurança do projeto (`SECURITY.md`) diz que:

- só a versão mais recente recebe correções; versões antigas não são atualizadas;
- vulnerabilidades devem ser relatadas por e-mail, e não em issues públicas;
- o projeto **não publica alertas de segurança separados nem registra CVEs**; as correções podem aparecer só nas notas de versão.

Na prática, quem instalou uma versão antiga e não acompanha as notas de versão pode continuar vulnerável sem saber.

### A responsabilidade do engenheiro

- **Tratar entradas do usuário como não confiáveis**, inclusive links e endereços de webhook, e validar o destino antes de fazer a requisição, como o projeto passou a fazer.
- **Manter padrões seguros.** A opção que desliga a proteção (`--allow-private-webhooks`) existe para casos legítimos, mas vem desligada. Quem a ativa precisa entender o risco.
- **Divulgar de forma responsável e clara.** Receber relatos em particular protege os usuários enquanto a correção não sai. Mas, depois de corrigir, avisar com clareza quem precisa atualizar também faz parte da responsabilidade. Num software instalado por milhares de pessoas, uma linha perdida nas notas de versão pode não ser suficiente.
- **Para nós:** a nossa implementação parte da `v0.31.0`, que já tem essas correções. Mas, como o projeto só corrige a versão mais nova, uma instância que ficasse parada nela deixaria de receber as próximas correções. Por isso registramos, nas limitações da implementação, que ela precisaria ser adaptada à versão atual antes de ser usada de verdade.

## 8.3 Inteligência artificial: acesso às notas, dados fora do servidor e revisão por robô

### O problema

A inteligência artificial aparece no Memos de três formas, cada uma com uma questão diferente:

1. **Assistentes de IA lendo as notas.** O servidor MCP (`server/mcp/`, registrado em `server/server.go`) permite que assistentes de IA leiam e criem memos em nome do usuário, usando um token de acesso pessoal. As notas podem conter informações pessoais ou de trabalho, e o usuário perde parte do controle sobre o que é lido e por quem.
2. **Dados saindo do servidor.** A transcrição de áudio (`server/api/v1/ai_service.go`) envia o áudio para provedores externos, como OpenAI e Gemini (`provider/ai/`). Isso contraria a principal promessa de um software *self-hosted*, de que "os seus dados ficam com você": quando o recurso está ativo, eles saem do servidor e passam a seguir as regras de outra empresa.
3. **Código revisado por robô.** Como mostramos na seção de processo (2.2), nos dois maiores PRs de 2026 (#6228, Spaces, e #6334, exportação) a única revisão registrada foi a do CodeRabbit, uma ferramenta automática de revisão por IA. O revisor humano foi indicado, mas a revisão ficou pendente, e o código foi integrado assim mesmo. No PR #6228, isso incluiu regras de autorização (quem pode ler cada memo).

### A responsabilidade do engenheiro

- **Pedir consentimento e informar.** Recursos que enviam dados para fora do servidor deveriam vir desligados e ser ativados conscientemente pelo administrador, com um aviso claro aos usuários de que o áudio será processado por um terceiro.
- **Limitar o acesso dos assistentes.** O acesso via MCP deveria respeitar as mesmas regras de visibilidade da interface, com tokens que possam ser revogados a qualquer momento.
- **Não terceirizar a responsabilidade para a IA.** Uma ferramenta de revisão automática ajuda, mas não substitui uma pessoa que entenda o impacto da mudança, principalmente em código de segurança e autorização. Quem aprova e integra continua responsável pelo que entra no projeto.
- **Para nós:** também usamos IA neste trabalho. Seguimos a mesma regra: toda afirmação vinda da IA foi conferida no repositório, cada integrante revisou o PR do outro (sem revisão automática), e o uso está registrado em [`08-ia/ia.md`](../08-ia/ia.md).