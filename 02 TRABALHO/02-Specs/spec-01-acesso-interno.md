# SPEC F1.1 — Acesso interno e permissões

**Fase:** 1 · **Resultado:** usuário autorizado vê apenas a carteira e os campos aprovados para seu papel.  
**Bloqueios:** B3 e definição de escopo por projeto/atribuição.

## Papéis iniciais

| Papel | Leitura de carteira | Detalhe de projeto | Operações manuais | Configurações |
|---|---|---|---|---|
| Administrador Thórus | Todos os projetos | Todos os projetos | Gerir usuários, atribuições e listas; CRUD de suporte; consultar auditoria | Sim; sem conectar/sincronizar serviços externos nesta fase |
| Atendimento/CS | Projetos atribuídos ou carteira aprovada | Dados de acompanhamento, tarefas e documentos internos | Criar/editar projetos conforme atribuição; tarefas e referências | Não |
| Engenharia/Projetista | Projetos atribuídos | Dados de projeto, fase, tarefa e definições | Criar/editar tarefas e propor alterações; aprovar apenas se designado | Não |
| Aprovações Legais | Projetos atribuídos à área | Eventos legais/tarefas autorizadas | Registrar e atualizar eventos legais e evidências | Não |
| Liderança | Todos os projetos-piloto aprovados | Todos os campos internos autorizados | Leitura; eventual edição somente se aprovada na matriz | Não |

A tabela é a proposta mínima de papéis para a Fase 1; responsável Thórus aprova ou ajusta antes de liberar usuários. Nenhum perfil externo/cliente existe nesta fase. O desenho deve permitir ajustar as regras sem duplicar verificações apenas na interface.

## Requisitos funcionais

1. Uma rota/ação protegida exige sessão autenticada e usuário ativo. Sem sessão, redirecionar ao fluxo de autenticação; usuário desativado perde acesso nas requisições seguintes.
2. O método de autenticação (Google Workspace SSO ou outro IdP) será escolhido por B3. Não armazenar senha Asana nem reutilizar token Asana como login da aplicação.
3. Cada consulta e operação no servidor verifica papel e conjunto de projetos permitidos. Ocultar botão no frontend é complementar; não substitui a autorização no endpoint.
4. Um usuário sem permissão não recebe campos nem objetos do projeto na resposta. Resposta de objeto fora do escopo deve ser equivalente a não encontrado, sem revelar nome/cliente.
5. Configurações de usuário/listas e mapeamentos futuros só podem ser alteradas por Administrador Thórus. Na Fase 1 não existe conexão Asana ativa nem painel de sincronização; nenhuma tela expõe token ou segredo.
6. Encerrar sessão revoga a sessão corrente. Desativar conta impede novas sessões e invalida as existentes dentro do limite suportado pela plataforma de autenticação.
7. Registrar eventos de auditoria: login/logout, falha de login (sem senha), acesso negado, mudança de papel/atribuição/configuração, criação/edição/arquivamento de projeto, mudança de tarefa/definição/evento legal/documento. Cada evento registra ator, ação, alvo, instante UTC e resultado; não registrar conteúdo de senha nem payload pessoal completo.
8. Exibir erros de autenticação de forma acionável, sem stack trace, token, GID sensível ou detalhe de configuração ao usuário final.

## Dados e regras

Identidade local deve guardar identificador estável do IdP, nome/e-mail corporativo quando permitido, estado ativo e papel. Atribuições de projeto referenciam o ID interno do projeto; GID Asana é opcional e não necessário para uso manual. A ausência de atribuição não concede acesso por padrão, exceto Admin/Liderança conforme decisão. Não inferir papel por domínio de e-mail, cargo no Asana ou coincidência de nome.

## Critérios de aceite verificáveis

- Cada papel de teste acessa apenas as telas, ações e projetos aprovados na matriz.
- Chamada direta ao endpoint com projeto alheio retorna sem dados; teste cobre lista e detalhe.
- Usuário desativado não consegue manter acesso em sessão previamente aberta.
- Alteração de papel/atribuição e ação administrativa ficam auditadas.
- Respostas e logs não expõem bearer token nem segredo.

## Provas necessárias

Matriz de papéis assinada, casos de acesso permitido/negado para cada papel, evidência de sessão invalidada, amostra de auditoria redigida e inspeção dos logs de erro.

## Perguntas de validação do cliente

Consulte e responda P8 no [questionário de levantamento](../05-Levantamento/06-Questionario-levantamento-cliente.md). Os passos para verificar APIs, campos e status estão nos apêndices do mesmo documento. Esta SPEC só deve ser fechada nos pontos afetados depois dessas respostas.
