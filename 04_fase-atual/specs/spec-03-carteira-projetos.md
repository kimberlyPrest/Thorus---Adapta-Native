# SPEC F1.3 — Interface completa e operação manual de projetos

**Fase:** 1<br>
**Status:** planejada<br>
**Dono:** Engenharia de produto<br>
**Origem no escopo:** seções 4.1 e 4.9; P5/P8/P9/P10<br>
**Degrau da solução:** construção mínima de interface web interna responsiva, usando os dados persistidos definidos na SPEC F1.2.

## Mapa de navegação

### Sidebar global

1. **Dashboard** — resumo acionável da carteira.
2. **Projetos** — lista/pesquisa e criação de projeto.
3. **Configurações** — somente Admin: usuários, papéis, listas de estados e mapeamentos futuros.
4. Área inferior: nome/papel do usuário, status de sessão e **Sair**.

Não colocar itens de detalhe de projeto na sidebar global. Ao entrar em um projeto, mostrar breadcrumb `Projetos > [Código/Nome] > [Aba]` e navegação secundária nas abas abaixo.

### Páginas e comportamento

| Página | Conteúdo e ações manuais | Acesso inicial |
|---|---|---|
| Entrar | Login escolhido para organização; erro, sessão expirada, usuário desativado e sem autorização com próximo passo claro | Todos os usuários internos cadastrados |
| Dashboard | Quantidade de projetos por fase/situação; tarefas vencidas/próximas; definições aguardando validação; eventos legais recentes; atividade recente. Cada cartão navega à lista filtrada que o compõe | Conforme carteira permitida |
| Projetos | Tabela paginada, busca, filtro combinado, ordenação, “Novo projeto”, abrir, editar e arquivar conforme papel | CS/Admin/Liderança; demais por atribuição |
| Novo/Editar projeto | Formulário para código, cliente, empreendimento, município/UF, escopo, fase, situação, datas, CS, responsável técnico, observações e membros. Campos obrigatórios são explicitamente marcados e configuráveis | Admin/CS; liderança se aprovado |
| Projeto — Visão geral | Identificação, contatos internos necessários, escopo, responsáveis, fase/status, datas, riscos/observações, próximos prazos e atividade recente | Membro atribuído; Admin/Liderança conforme matriz |
| Projeto — Tarefas e fases | Fases ordenadas; criar/editar/concluir/reativar tarefa; responsável, prazo, prioridade, estado, descrição e observação. Exibir vencida/próxima sem enviar alertas externos | CS/Engenharia/Legais de acordo com atribuição |
| Projeto — Definições e alterações | Definições atuais por disciplina; iniciar solicitação com valor proposto, motivo, solicitante, origem e impacto informado; papel autorizado aprova/recusa/cancela; comparar versão atual e anterior | Engenharia; aprovadores definidos por papel |
| Projeto — Legais e marcos | Registrar protocolo, exigência, parecer, aprovação ou tipo configurado; estado, número, data, responsável, comentário e link/evidência. Mudança de estado guarda autor e data | Legais/CS; leitura dos demais autorizados |
| Projeto — Documentos | Lista de referências cadastradas manualmente: nome, categoria, URL, observação, data e visibilidade. Adicionar/editar/remover referência com confirmação; abrir link informado em nova aba | Membros do projeto; visibilidade interna |
| Projeto — Atividade | Linha do tempo do que ocorreu: entidade/registro, ação, autor, instante local e origem manual; filtrar por tipo/período | Leitura conforme projeto e sensibilidade |
| Configurações — Usuários e papéis | Ativar/desativar conta, papel e projetos atribuídos; alterações auditadas; não cadastrar senha do usuário no sistema se IdP externo for usado | Admin |
| Configurações — Listas | Manter valores permitidos de fases, situações, disciplinas, prioridades e tipos de evento; alterações não apagam o valor histórico já usado | Admin |
| Configurações — Integrações planejadas | Mostrar campos externos mapeados, revisão/pendências e aviso “Conexão prevista para Fase 2”; preparar GID, workspace e permalink, todos opcionais; sem ação “conectar”, token ou chamada de rede | Admin somente |

## Dashboard e carteira

- Contadores são calculados dos registros autorizados do banco na hora em que a página é consultada; apresentar “atualizado em” como horário da consulta, não como sincronização externa.
- Os cards de atrasos usam somente prazo presente e estado não concluído; zonas de data são definidas em São Paulo.
- Busca por código, projeto, empreendimento e cliente permitido; texto não pesquisa conteúdo de projeto sem autorização.
- Filtros: cliente, empreendimento, fase, situação, CS, responsável técnico, município/UF, tarefas vencidas, alterações pendentes, eventos legais e arquivado/ativo, conforme permissões.
- Lista oferece paginação, ordenação visível e opção de limpar todos os filtros. Não há resultados e nenhum projeto são estados diferentes.
- Ação de arquivar preserva o histórico e pede confirmação; não excluir definitivamente por ação cotidiana.

## Regras dos formulários/fluxos

- Validar obrigatoriedade, tamanho, formato de e-mail/URL, datas coerentes e existência de referência selecionada no servidor e na interface; explicar o campo sem jargão.
- Salvar informa sucesso apenas após confirmação de persistência. Em falha, preservar campos preenchidos para correção/reenvio.
- Em conflitos de edição, alertar que o registro mudou desde a abertura e permitir recarregar/comparar antes de substituir.
- Ao arquivar projeto, concluir/reatribuir tarefa, aprovar definição ou mudar estado legal, mostrar o efeito antes da confirmação. Mudança aprovada em definição cria nova versão.
- Toda ação manual significativa registra ator, timestamp UTC, tipo da ação, entidade e alteração mínima necessária; evitar gravar texto pessoal integral em logs genéricos.
- Documentos aceitam apenas metadados/link. A Fase 1 não faz upload, download, preview de binário, criação de pasta nem verificação automática da ACL do Drive.
- Nenhum registro pode ser criado/modificado automaticamente a partir de Asana/Gemini/WhatsApp nesta fase.

## UI/UX e acessibilidade

- Interface em português brasileiro; sidebar com ícone e texto, item ativo claro, rótulos persistentes em formulários e breadcrumbs de contexto.
- Desktop como superfície principal e adaptação para tablet/celular: tabelas viram cartões/lista com campos essenciais; ações não podem depender de hover.
- Hierarquia visual prioriza situação, responsável e próximo prazo. Usar badge com texto e cor, sem cor como único sinal.
- Estados obrigatórios em páginas de dados: carregando, vazio (com ação de criar quando permitido), sem resultado, erro recuperável, acesso negado e sucesso após gravar.
- Formulário divide campos em grupos curtos e mantém dados em erro. Ação primária única e secundárias claramente nomeadas; confirmação para ações com impacto.
- Acessível por teclado, foco visível, ordem lógica, contraste legível, nome acessível em controles e mensagens de validação associadas ao campo.
- Datas exibidas em formato brasileiro e fuso `America/Sao_Paulo`; timestamps sem hora na origem não ganham horário inventado.
- Conteúdo livre é tratado como texto e não executa HTML/script. URL é validada e identificada como externa antes de abrir.

## Critérios de aceite

- [ ] **CA-F1.3-01:** Um usuário CS cria projeto, cadastra responsável e tarefa, depois sai/entra e encontra os dados persistidos.
- [ ] **CA-F1.3-02:** Engenharia cria solicitação de alteração; aprovador autorizado decide; visão geral mostra definição atual e atividade conserva a versão anterior.
- [ ] **CA-F1.3-03:** Legais registra evento e evidência URL; consegue corrigir uma informação mantendo rastreabilidade conforme regra aprovada.
- [ ] **CA-F1.3-04:** Dashboard e filtros levam à lista coerente e nunca incluem projetos fora do acesso.
- [ ] **CA-F1.3-05:** Cada página implementa seus estados vazio, erro, carregamento e sucesso; formulários apresentam validação compreensível.
- [ ] **CA-F1.3-06:** Sidebar e detalhe permanecem navegáveis por teclado e viewport estreita; estado atual não depende apenas de cor.
- [ ] **CA-F1.3-07:** Um link externo manual abre o endereço informado; sistema não acessa API de Asana/Drive.
- [ ] **CA-F1.3-08:** Usuário sem permissão não consegue ler/gravar via URL direta, endpoint ou ID de registro.

## Perguntas de validação do cliente

Consulte P1, P2, P5, P8, P9 e P10 no [questionário de levantamento](../../03_documentos/06-Questionario-levantamento-cliente.md). P1/P2 informam dados e mapeamentos futuros; P5/P9 estados/aprovadores; P8 papéis; P10 baseline. Instruções de API são para preparar Fase 2, sem conectar serviços na Fase 1.


## Contexto e decisões fechadas

- **Estado atual:** a equipe consulta múltiplas ferramentas para saber estado e histórico do projeto; o novo banco começa manual.
- **Estado desejado:** navegação interna única e responsiva com projetos, tarefas, definições versionadas, eventos legais, referências documentais e atividade.
- **Decisões já fechadas:** labels em português; sidebar global Dashboard/Projetos/Configurações; seções específicas como abas no detalhe; CRUD manual; sem integrações nesta fase.
- **Bloqueios:** confirmar campos obrigatórios, papéis e vocabulários com P1/P2/P5/P8/P9; listas opcionais mantêm default não informado.

## Resultado observável

CS cadastra projeto, cria tarefa e acompanha prazo; Engenharia registra alteração e aprovação; Legais registra protocolo; os dados aparecem no dashboard/detalhe após sair e entrar novamente, sem API externa.

## Limites e dependências

- **Inclui:** Entrar, Dashboard, Projetos, Novo/Editar, detalhe com seis abas, Configurações e fluxos manuais previstos nas tabelas desta SPEC.
- **Fora de escopo:** portal cliente, upload Drive, sync Asana, WhatsApp/email automático, Gemini e relatório enviado.
- **Entradas/pré-condições:** F1.1 autenticação/autorização e F1.2 persistência; papéis P8; termos/estados P5/P9.
- **Saídas:** páginas navegáveis, registros CRUD manuais, filtros e estados de UI.
- **Donos:** Engenharia e Produto; CS/Engenharia/Legais validam seus fluxos; Admin mantém configurações.
- **Superfícies afetadas:** sidebar, dashboard, listagem/formulários e tabs do projeto, configurações, endpoints de leitura/escrita.
- **Risco/plano B:** se vocabulário não confirmado, manter opção configurável e mostrar “Não informado”; não congelar status presumido como oficial.
- **Rollback:** desativar rota ou controle com falha sem apagar registros; migração de UI não altera histórico. Correção de dado cria evento.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| UI ↔ API local ↔ banco | Banco definido por F1.2 para registro manual | projects, memberships, tasks, definitions/versions, legal_events, document_references, activity_events | Guard de F1.1 em cada endpoint | Paginado server-side; submit repetido usa ID/idempotência da ação | Erro mantém formulário; sucesso só após persistência |
| Referência URL → navegador | URL fornecida pelo usuário | scheme/host/path e rótulo/categoria visíveis | Projeto e visibilidade interna | Sem fetch server-side; clique manual | URL inválida ou externa recebe alerta/validação |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-F1.3-01 | Projeto ativo dentro da permissão | Aparece em lista, contagem e busca | Arquivado só aparece com filtro explícito | Páginas e matriz F1.1 |
| RN-F1.3-02 | Definição aprovada é substituída | Nova versão vira atual; anterior preservada | Usuário sem papel de aprovador não altera vigente | Seção Definições e alterações/P9 |
| RN-F1.3-03 | Prazo menor que data local e tarefa não concluída | Mostrar atrasada no dashboard e filtro | Sem prazo não entra como atrasada | Tarefas e dashboard |
| RN-F1.3-04 | URL/documento inserido manualmente | Guardar metadado e link; não afirmar permissão Drive | Sem upload ou ACL na Fase 1 | Seção Documentos |

## Fluxo e regras

1. Entrar → dashboard com indicadores clicáveis → projetos com busca, filtros e paginação.
2. “Novo projeto” grava campos obrigatórios; depois exibe overview e abas do mesmo projeto.
3. Usuário cria tarefa/fase, definição/solicitação, evento legal ou referência documental conforme seu papel.
4. Mudança de estado/definição crítica pede confirmação, persiste e aparece em atividade/auditoria.
5. Falha de rede/aplicação ou permissão exibe estado recuperável; não some com formulário nem declara sucesso.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | CS cria projeto, tarefa e prazo; Engenharia registra alteração | Todos os dados permanecem após nova sessão; dashboard reflete filtros | Se gravação falhar, preservar formulário e exibir erro |
| Limite | Sem tarefas/documentos ou sem resultado de filtro | Estado vazio específico com ação permitida e limpar filtros | Sem acesso, mostrar tela negada em vez de lista vazia |
| Falha | Data inválida, URL inválida, conflito de edição ou endpoint indisponível | Campo/registro não é corrompido; erro explica próxima ação | Corrigir valor, recarregar/comparar versão ou tentar novamente |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** páginas e papéis desta SPEC, schema F1.2 e matriz P8/P5/P9.
2. **Alterar somente:** shell, páginas e CRUD manual; não construir conector externo.
3. **Não alterar:** taxonomia legal/técnica não aprovada, permissão, esquema sem migração aprovada, publicação externa.
4. **Executar nesta ordem:** shell/sidebar; dashboard e carteira; overview; abas; configurações e estados/validações.
5. **Parar e pedir validação quando:** campo obrigatório, status oficial, papel de aprovação ou compartilhamento externo não estiver decidido.
6. **Estado válido ao parar:** dados persistidos; rotas autorizadas; formulários em erro mantêm informação; nenhuma integração ativa.

## Checklist de execução

- [ ] Sidebar, breadcrumbs e páginas listadas nesta SPEC estão acessíveis.
- [ ] CRUD manual e filtros funcionam em ao menos um projeto fictício.
- [ ] Vazio, carregamento, erro, sucesso, validação e acesso negado têm estados explícitos.
- [ ] Data, estado e autoria aparecem corretamente na atividade.
- [ ] Teste de viewport estreito/teclado/foco foi evidenciado.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | Navegar sem páginas/CRUD; acesso não filtrado por projeto | Teste de rota/componentes e endpoint com usuários fixture | Página ausente ou consulta expõe projeto fora do vínculo | Relatório de falha focado |
| GREEN | Criar projeto e percorrer tarefa, definição, evento, documento e atividade | Testes de integração para formulário/API/banco e fluxo UI automatizado | CRUD persiste; permissões e dashboard refletem estado | Relatório de fluxo e screenshots sem PII |
| REFACTOR/REGRESSÃO | Filtro vazio, API indisponível, viewport estreito, teclado, URL inválida | Rodar suíte focal e roteiro manual de acessibilidade/estados | Recuperação compreensível sem vazamento/perda | Evidências por estado e viewport |

**Dados/fixtures:** projeto fictício, CS/Engenharia/Legais/Admin, tarefas em três estados, definição versões 1/2 e eventos de exemplo.<br>
**Caminhos de erro obrigatórios:** vazio, sem resultado, sem permissão, validação, conflito, indisponibilidade e link inválido.<br>
**Evidência exigida:** roteiro de demonstração e matriz página × papel × estado.

## Handoff e operação

- **Como demonstrar:** criar um projeto; adicionar tarefa, definição, evento e link; confirmar os indicadores no dashboard.
- **Como operar depois:** CS/Engenharia/Legais atualizam seus registros pela aplicação; Admin mantém listas e usuários.
- **Como monitorar:** falhas de salvamento, erros de rota e operações sem auditoria.
- **Pendência conhecida:** dados de produção e vocabulários finais dependem de P1/P2/P5/P8/P9.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| F1-01 | Confirmar campos mínimos e vocabulários para piloto | Produto | F1.3 | Dicionário de campos/tipos/regras e registro fictício aprovados | Contexto e decisões; Novo/Editar projeto | Dicionário e amostra preenchida | Responsável CS disponível; P5/P8/P9 | Planejada |
| F1-05 | Construir shell, sidebar, breadcrumbs e páginas | Engenharia | F1.3 | Páginas/abas previstas acessíveis e responsivas | Mapa de navegação; UI/UX | Roteiro por página/papel | F1.1,F1.2 | Planejada |
| F1-06 | Implementar Dashboard/carteira e CRUD de projeto | Engenharia | F1.3 | Criar/editar/arquivar/filtrar projeto respeitando acesso | Dashboard e carteira; RN-F1.3-01/03 | Demonstração CRUD + dashboard | F1-01,F1-05,F1-15 | Planejada |
| F1-07 | Implementar tarefas e fases manuais | Engenharia | F1.3 | Tarefa mantém responsável/fase/estado/prazo | Tarefas e fases; RN-F1.3-03 | Cenários prazo/estado/vazio | F1-06, listas aprovadas | Planejada |
| F1-11 | Implementar definições e alterações versionadas | Engenharia | F1.3 | Aprovador muda vigente sem remover a anterior | Definições e alterações; RN-F1.3-02 | Histórico comparável | F1-06,P9 | Planejada |
| F1-12 | Implementar eventos legais e marcos | Engenharia | F1.3 | Evento guarda tipo/responsável/data/estado/evidência | Legais e marcos; fluxo | Cenário por tipo P5 | F1-06,P5 | Planejada |
| F1-13 | Implementar referências de documentos e atividade | Engenharia | F1.3 | URL manual e histórico visíveis; sem upload | Documentos; Atividade; RN-F1.3-04 | Registro e auditoria consultáveis | F1-06 | Planejada |
| F1-16 | Implementar validação, estados de UI e acessibilidade | Engenharia | F1.3 | Estados distinguíveis, formulário preservado e teclado funcional | UI/UX; Critérios de aceite; TDD regressão | Matriz de estados e capturas | F1-05,F1-06,F1-07,F1-11–F1-14 | Planejada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
