# SPEC F1.1 — Acesso interno e permissões

**Fase:** 1<br>
**Status:** planejada<br>
**Dono:** Engenharia de produto<br>
**Origem no escopo:** seção 3 (Pessoas e permissões), seção 4.9, P8<br>
**Degrau da solução:** reuso do IdP corporativo escolhido, com autorização local por papel e projeto.

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

## Critérios de aceite

- [ ] **CA-F1.1-01:** Cada papel de teste acessa apenas as telas, ações e projetos aprovados na matriz.
- [ ] **CA-F1.1-02:** Chamada direta ao endpoint com projeto alheio retorna sem dados; teste cobre lista e detalhe.
- [ ] **CA-F1.1-03:** Usuário desativado não consegue manter acesso em sessão previamente aberta.
- [ ] **CA-F1.1-04:** Alteração de papel/atribuição e ação administrativa ficam auditadas.
- [ ] **CA-F1.1-05:** Respostas e logs não expõem bearer token nem segredo.

## Provas necessárias

Matriz de papéis assinada, casos de acesso permitido/negado para cada papel, evidência de sessão invalidada, amostra de auditoria redigida e inspeção dos logs de erro.

## Perguntas de validação do cliente

Consulte e responda P8 no [questionário de levantamento](../../03_documentos/06-Questionario-levantamento-cliente.md). Os passos para verificar APIs, campos e status estão nos apêndices do mesmo documento. Esta SPEC só deve ser fechada nos pontos afetados depois dessas respostas.


## Contexto e decisões fechadas

- **Estado atual:** não existe no MVP uma aplicação com usuários e carteira própria protegidos por autorização central.
- **Estado desejado:** identidade corporativa autenticada, acesso por papel/projeto e autorização em servidor em todas as operações.
- **Decisões já fechadas:** sem usuários clientes na Fase 1; GID Asana não é necessário para atribuição; acesso não se concede por domínio de e-mail.
- **Bloqueios:** método de login (B3) e matriz de visibilidade (P8) devem ser aprovados antes de cadastrar usuários reais; sem aprovação, manter ambiente de demonstração.

## Resultado observável

CS e Engenharia entram e veem somente os projetos autorizados. Tentativa de abrir projeto alheio por URL ou endpoint não retorna nome, cliente, campos nem contagem.

## Limites e dependências

- **Inclui:** sessão, papéis, atribuições, validação de leitura/escrita, logout/desativação e auditoria de acesso.
- **Fora de escopo:** portal externo, integração de login com Asana, autenticação construída do zero sem decisão, privilégio inferido.
- **Entradas e pré-condições:** IdP escolhido; usuários de teste; papel e atribuição por projeto aprovados por Thórus.
- **Saídas/artefatos:** matriz papel × página × ação; configuração do IdP; evidências de acesso e negação.
- **Atores e permissões mínimas:** Admin, CS, Engenharia, Legais e Liderança conforme a tabela de Papéis iniciais; sem perfil cliente.
- **Superfícies afetadas:** entrada, sessão, middleware/endpoints, carteira, detalhe e Configurações > Usuários.
- **Risco e plano B:** se o IdP escolhido não funcionar no ambiente de teste, pausar login real e demonstrar com provedor já suportado; nunca criar bypass.
- **Rollback:** desativar integração/configuração de login ou papel, invalidar sessões quando suportado e manter trilha de auditoria.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| IdP → sessão local | IdP para identidade; banco local para papel e atribuição | ID estável do sujeito, estado ativo e claims mínimos aprovados | Provedor escolhido em B3; sessão validada pelo servidor | Callback protegido conforme IdP; renovação conforme mecanismo suportado | Sessão vencida/inativa volta ao login; falha não concede acesso |
| Aplicação → registros | Banco local | user_id, project_id, role, ação permitida | Guard de servidor em toda leitura/escrita | Consulta não pode contornar atribuição por ID direto | Ocultar existência de objeto não autorizado |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-F1.1-01 | Usuário sem atribuição pede projeto | Negar leitura e mutação sem revelar metadados | Admin/Liderança somente se P8 aprovar carteira transversal | P8, matriz de acesso |
| RN-F1.1-02 | Papel ou atribuição muda | Próxima requisição aplica regra nova e gera auditoria | Revogação de sessão conforme limite do IdP | Requisitos 5–7 desta SPEC |

## Fluxo e regras

1. Usuário autentica no IdP aprovado; aplicação valida identidade e estado ativo.
2. Servidor resolve papéis/atribuições locais antes de retornar dados.
3. Mutação valida autorização, grava a mudança e registra auditoria de forma consistente.
4. Logout encerra sessão; desativação impede novos acessos no limite suportado pelo IdP.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Usuário ativo atribuído | Carteira autorizada aparece e ação permitida persiste | Reautenticar após sessão vencida sem apagar formulário |
| Limite | Usuário acessa URL de projeto sem vínculo | Sem conteúdo nem confirmação da existência | Retornar à carteira e registrar acesso negado |
| Falha | IdP indisponível ou usuário desativado | Não criar sessão | Orientar contato Admin; sem fallback privilegiado |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** matriz de papéis desta SPEC, resposta P8 e decisão B3.
2. **Alterar somente:** sessão interna, guards, papéis, atribuições e auditoria de acesso.
3. **Não alterar:** permissões do portal futuro, APIs Asana/Drive ou regras de publicação externa.
4. **Executar nesta ordem:** autenticar; proteger endpoints; conectar páginas aos guards; validar logout/desativação.
5. **Parar e pedir validação quando:** P8/B3 indefinida, campo pessoal sem finalidade ou nova exceção de acesso transversal.
6. **Estado válido ao parar:** Admin de teste entra; todas as rotas protegidas negam acesso anônimo; nenhum cliente externo existe.

## Checklist de execução

- [ ] IdP e matriz aprovados.
- [ ] Lista, detalhe e mutação validados por papéis permitido e negado.
- [ ] Logout, expiração e desativação validados.
- [ ] Logs não contêm segredo nem PII desnecessária.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | Acesso anônimo a carteira, detalhe e mutação | Teste HTTP automatizado dos três endpoints sem sessão | Resposta 401/403 sem payload de projeto | Relatório de integração auth |
| GREEN | CS atribuído ao projeto A e não atribuído ao B | Teste com duas fixtures e chamadas lista/detalhe/escrita | A autorizado; B negado sem metadado | Relatório por rota e papel |
| REFACTOR/REGRESSÃO | Desativar conta com sessão, alterar vínculo e sair | Cenário automatizado e conferência da auditoria | Novas requisições perdem acesso; evento registra alteração | Log redigido e resultado de regressão |

**Dados/fixtures:** Admin, CS, Engenharia, projetos A/B, usuário sem vínculo e usuário desativado.<br>
**Caminhos de erro obrigatórios:** sessão ausente/vencida, IdP indisponível, papel insuficiente e projeto alheio.<br>
**Evidência exigida:** matriz aprovada e relatório de lista/detalhe/mutação.

## Handoff e operação

- **Como demonstrar:** comparar carteiras de CS e Engenharia e tentar abrir projeto sem atribuição.
- **Como operar depois:** Admin mantém usuários, estado ativo, papéis e atribuições.
- **Como monitorar:** falhas de login, acessos negados e mudanças de permissão na auditoria.
- **Pendência conhecida:** decisão B3 e confirmação P8.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| F1-02 | Aprovar login, papéis e atribuições | Thórus Admin | F1.1 | Matriz cobre os 5 papéis, páginas e ações | Papéis iniciais; RN-F1.1-01/02 | Matriz papel × página × ação aprovada | B3/P8 respondidas | Planejada |
| F1-04 | Implementar sessão e autorização no servidor | Engenharia | F1.1 | Lista, detalhe e mutação negam dados fora da atribuição | Requisitos funcionais 1–8; TDD RED/GREEN | Relatório auth redigido | F1-02 e IdP de teste | Planejada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
