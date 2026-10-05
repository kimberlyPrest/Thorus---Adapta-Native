# SPEC-1-008 — Visão geral, equipe e arquivamento do projeto

**Fase:** 1  
**Status:** planejada  
**Dono:** Engenharia de produto (execução) e Thórus (validação do fluxo)  
**Origem no escopo:** seção 4.1 e 4.9; rota `/projetos/{id}`  
**Degrau da solução:** construção mínima da tela e da regra local necessárias para esta operação funcionar manualmente.

## Contexto e decisões fechadas

- **Estado atual:** não existe aplicativo executável neste repositório; a SPEC antiga correspondente está registrada no mapa de migração, commit `a359cdc`.
- **Estado desejado:** uma pessoa com o papel indicado completa a operação em `/projetos/{id}` e encontra o resultado persistido, autorizado e auditável.
- **Decisões já fechadas:** Fase 1 é manual; dados próprios ficam no banco C2; auth e grants C3/C4; UI D1–D5; nenhum conector de negócio externo nesta fase.
- **Bloqueios/dependências:** B1 para implementação de código; B3 para autenticação real. P8 para liberar usuários reais. Dependências técnicas de domínio: SPEC-1-001, SPEC-1-002, SPEC-1-004, SPEC-1-007

## Perguntas de validação do cliente

| Perguntas | O que confirmar / como responder | Referência |
|---|---|---|
| P1, P8 e P9: equipe por projeto, aprovadores e acesso atribuído/transversal. | Responder marcando opções ou com uma frase curta. Se não souber, indicar quem confirma. Para Asana, anotar nomes/valores e usar somente GET; não alterar dados nem enviar token. | [Questionário P1–P10](../../03_documentos/06-Questionario-levantamento-cliente.md) |

## Resultado observável

Arquivar confirma suspensão de gravação; Reativar recupera operação sem excluir dados.

## Limites e dependências

- **Inclui:** rota `/projetos/{id}`, leitura/mutação desta SPEC, persistência pertinente, grants de servidor, estados D3, activity C3 e evidência abaixo.
- **Fora de escopo:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
- **Entradas e pré-condições:** C2–C5, UI D1–D5, grants descritos abaixo; B1 para implementação de código; B3 para autenticação real. P8 para liberar usuários reais.
- **Saídas/artefatos:** página funcional, response envelope C3, migration/constraint necessários, eventos de auditoria, evidências CA.
- **Dependências e responsáveis:** dependências listadas no cabeçalho; Produto/Engenharia fecha B1; Thórus fecha perguntas P referenciadas.
- **Atores e permissões mínimas:** projects.read; editar projects.update; equipe projects.manage_team; arquivo projects.archive. Aprovadores técnicos seguem definitions.decide + projeto/disciplina.
- **Superfícies/arquivos/configurações afetadas:** rota, componentes próprios, endpoint/handler, tabelas C2 citadas, autorização C4 e audit event.
- **Risco e plano B:** ocultação visual não protege dados; aplicar a mesma decisão no servidor. Pendência de vocabulário usa somente fixture identificada até resposta P.
- **Rollback ou reversão:** tela pode suspender ação sem apagar histórico; migration aditiva/reversível; cancelamento preserva registro e emite auditoria.

## Página e comportamento detalhado

URL/rota: `/projetos/{id}`. Base visual única: [Contrato de interface](01-contrato-interface.md).

| Área | Conteúdo e interação |
|---|---|
| Header | Código/nome, cliente, fase/situação; Editar e menu Arquivar quando permitido |
| Resumo | Escopo, localização, datas, CS/responsável técnico e campos não informados; link às abas por contagem somente autorizada |
| Equipe | Lista nome/perfil e estado; Gerenciar equipe abre drawer de usuários ativos/seleções |
| Aprovadores | Dentro do drawer equipe, por disciplina selecionar membro com definitions.decide; aviso se sem aprovador |
| Arquivo | Modal com nome e “Dados/histórico preservados; novas mutações ficam suspensas”; visão arquivada leitura, Reativar autorizado |
| Sem autorização | 404 genérico sem header/cliente; não mostrar resumo parcialmente carregado |

## Dados de entrada e saída

| Campo | Regra |
|---|---|
| project header/summary | C2; sem duplicar versões definidas na aba técnica |
| memberships[] | Lista de usuários existentes, ativos; removidos desativam vínculo, não autoria |
| approvers[] | userId+disciplineId; usuário membro ativo e decide concedido |
| archivedAt/expectedVersion | Só server define instante; controle de conflito em todas ações |

## Dados e integrações

UI chama somente API local C3; tabelas respeitam FKs/índices C2. Nenhuma API Asana/Drive é chamada nesta rota.

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Erro |
|---|---|---|---|---|---|
| Tela `/projetos/{id}` ↔ API local ↔ banco | Banco da aplicação | Campos listados em Dados de entrada; versão/C2 | projects.read; editar projects.update; equipe projects.manage_team; arquivo projects.archive. Aprovadores técnicos seguem definitions.decide + projeto/disciplina. | C3: paginação, Idempotency-Key em escrita, expectedVersion em edição; sem retry automático de PATCH | Envelope C3, manter rascunho, correlationId |
| API local ↔ activity_events | Banco na mesma transação | ator, recurso, ação, alvo, instante UTC e resumo seguro | Herda grant da ação | Transação atômica; chave impede repetição de evento | Se log falhar, cancelar mutação e devolver erro C3 |

### Regras de negócio e regras de dados

Arquivo bloqueia criação/edição/transições nas abas; leitura continua se grant read e vínculo permitirem.
Remover usuário da equipe revoga próximos acessos ao projeto; não recalcula autor histórico.
Não remover último membro de projeto com escopo assigned sem outro responsável autorizado (Admin all é via recuperação de acesso).
Nova política que tira definitions.decide torna designação inelegível; detalhe sinaliza Sem aprovador válido e decisão bloqueia até corrigir.

## API local desta entrega

- GET /api/projects/{id} e /header → resumo/membros/capacidades autorizados.
- PUT /api/projects/{id}/team {members,approvers,expectedVersion} → memberships/designações + evento atômicos.
- POST /api/projects/{id}/archive ou /restore {expectedVersion} → muda archived_at e auditoria.

## Fluxo principal e caminhos de erro

1. Abrir Visão geral, conferir responsáveis e cadastrar equipe pelo drawer.
2. Designar aprovador por disciplina e comprovar que outro usuário não recebe autoridade.
3. Arquivar confirma suspensão de gravação; Reativar recupera operação sem excluir dados.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Abrir Visão geral, conferir responsáveis e cadastrar equipe pelo drawer. | Resumo/header/equipe refletem os dados persistidos e não mostram recursos sem read. | Erro de gravação mantém formulário segundo D3. |
| Permissão/limite | projects.read; editar projects.update; equipe projects.manage_team; arquivo projects.archive. Aprovadores técnicos seguem definitions.decide + projeto/disciplina. | Gestão de membros/aprovadores muda acesso de servidor e preserva autoria histórica. | Responder 401/403/404 conforme C3, sem vazamento. |
| Falha/concorrência | Arquivar mantém histórico e bloqueia mutações de todas abas; restore manual reabre sem perder dados. | Dados permanecem íntegros, sem sucesso falso. | 409/422/503 recuperável; campos mantidos e nenhuma alteração parcial. |

## Critérios de aceite

- [ ] **SPEC-1-008-CA-01:** Resumo/header/equipe refletem os dados persistidos e não mostram recursos sem read.
- [ ] **SPEC-1-008-CA-02:** Gestão de membros/aprovadores muda acesso de servidor e preserva autoria histórica.
- [ ] **SPEC-1-008-CA-03:** Arquivar mantém histórico e bloqueia mutações de todas abas; restore manual reabre sem perder dados.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | SPEC-1-008-CA-01 e SPEC-1-008-CA-03 com fixture C5 | Antes da implementação, executar a ação de UI/API descrita nesta SPEC para os critérios principal, negado e conflito | Falha por ausência da regra/comportamento, sem falha de fixture; registrar status/estado observado | Saída do cenário e estado inicial fictício |
| GREEN | Todos os CA desta SPEC | Implementar a rota ponta a ponta; repetir UI/API, reler do banco e tentar request sem grant/com versão obsoleta | Cada CA passa; mutação e evento são atômicos; erro mantém rascunho | Captura por estado, resposta C3 redigida, releitura e evento |
| REFACTOR | Todos os CA desta SPEC | Reexecutar fluxo, teclado e viewports 375/768/1440; aplicar o runner escolhido em B1 quando houver código | Sem regressão; critérios continuam binários; autorização server-side e dados íntegros | Relatório focal e evidências por CA |

**Dados/fixtures:** [Contrato de dados, C5]; adicionar somente as entidades exigidas pela tabela Dados de entrada desta SPEC.  
**Caminhos de erro obrigatórios:** Arquivar confirma suspensão de gravação; Reativar recupera operação sem excluir dados. Erro de permissão, referência inválida, conflito de versão e falha de banco conforme C3.  
**Evidência exigida:** um artefato por critério; captura de página com dado fictício, API response sem segredo, verificação de persistência/autorização.

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** [contrato visual](01-contrato-interface.md), [dados/API](02-contrato-dados-api.md), seção de limite/permissão desta SPEC e SPEC-1-001, SPEC-1-002, SPEC-1-004, SPEC-1-007.
2. **Alterar somente:** rota `/projetos/{id}`, componente, handler/endpoint e persistência deste fluxo.
3. **Não alterar:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
4. **Executar nesta ordem:** migration/contrato; leitura autorizada; ação principal e commit; estado de erro; teclado/mobile; captura de evidência.
5. **Parar e pedir validação quando:** B1 para implementação de código; B3 para autenticação real. P8 para liberar usuários reais.
6. **Estado válido ao parar:** rota principal não grava parcial; sessão/permissões protegem resposta; dados existentes e histórico continuam íntegros.

## Checklist de execução

- [ ] Grant e lista de campos confirmados; fixture mínima tem chave/teste repetível.
- [ ] Caminho principal liga tela, servidor, banco e evento de auditoria na mesma entrega.
- [ ] Vazio, erro, inválido, negado, concorrência e sucesso aplicam D3 na rota.
- [ ] Teclado/foco/viewport e evidências por CA verificados.
- [ ] Thórus confere o fluxo com dados fictícios; pendência de produto registrada.

## Handoff e operação

- **Como demonstrar:** Arquivar confirma suspensão de gravação; Reativar recupera operação sem excluir dados.
- **Como operar depois:** Thórus opera pela rota indicada usando os grants C4.
- **Como monitorar:** correlationId, falha de gravação, 401/403, conflito e evento da rota; nenhum log guarda segredo.
- **Pendência conhecida:** B1 para implementação de código; B3 para autenticação real. P8 para liberar usuários reais.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Subseção exata | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| F1-24 | Gerenciar equipe e aprovadores na Visão geral do projeto | Thorus | SPEC-1-008 | Admin/gestor atribui membro e aprovador somente em disciplina compatível e ativa acesso correspondente. | Página e comportamento detalhado | SPEC-1-008-CA-01; cenário de UI/API, releitura e evidência | Drawer equipe, requisição PUT, acesso membro, negado não membro. | P8/P9 e F1-02/F1-19. | Parar se papel de aprovador vier só do perfil Engenharia ou vínculo ficar órfão. | Planejada |
| F1-37 | Aplicar atribuição, leitura e bloqueio em projeto arquivado | Thorus | SPEC-1-008 | Membro removido perde acesso na requisição seguinte; projeto arquivado mantém leitura autorizada e nega escrita nas seis abas. | API local; Regras de negócio | SPEC-1-008-CA-02; cenário de UI/API, releitura e evidência | Admin/CS/membro removido, 404/403, reativação e link de histórico. | F1-24, F1-19 e C4. | Parar se ID direto bypassar memberships/arquivo ou excluir dado. | Planejada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
