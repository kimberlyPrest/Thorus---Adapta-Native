# SPEC-1-015 — Dashboard acionável com métricas e detalhes

**Fase:** 1  
**Status:** planejada  
**Dono:** Engenharia de produto (execução) e Thórus (validação do fluxo)  
**Origem no escopo:** seção 4.1 e 4.9; rota `/dashboard e /dashboard/{tarefas|alteracoes}`  
**Degrau da solução:** construção mínima da tela e da regra local necessárias para esta operação funcionar manualmente.

## Contexto e decisões fechadas

- **Estado atual:** não existe aplicativo executável neste repositório; a SPEC antiga correspondente está registrada no mapa de migração, commit `a359cdc`.
- **Estado desejado:** uma pessoa com o papel indicado completa a operação em `/dashboard e /dashboard/{tarefas|alteracoes}` e encontra o resultado persistido, autorizado e auditável.
- **Decisões já fechadas:** Fase 1 é manual; dados próprios ficam no banco C2; auth e grants C3/C4; UI D1–D5; nenhum conector de negócio externo nesta fase.
- **Bloqueios/dependências:** B1 para implementação de código; B3 para autenticação real. P8 para liberar usuários reais. Dependências técnicas de domínio: SPEC-1-001, SPEC-1-002, SPEC-1-004, SPEC-1-006, SPEC-1-009, SPEC-1-011, SPEC-1-012, SPEC-1-014

## Perguntas de validação do cliente

| Perguntas | O que confirmar / como responder | Referência |
|---|---|---|
| P6 e P10: indicadores de acompanhamento, frequência e medição antes/depois. | Responder marcando opções ou com uma frase curta. Se não souber, indicar quem confirma. Para Asana, anotar nomes/valores e usar somente GET; não alterar dados nem enviar token. | [Questionário P1–P10](../../03_documentos/06-Questionario-levantamento-cliente.md) |

## Resultado observável

Mudar tarefa, atualizar e comparar com usuário que não pode ler tarefas.

## Limites e dependências

- **Inclui:** rota `/dashboard e /dashboard/{tarefas|alteracoes}`, leitura/mutação desta SPEC, persistência pertinente, grants de servidor, estados D3, activity C3 e evidência abaixo.
- **Fora de escopo:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
- **Entradas e pré-condições:** C2–C5, UI D1–D5, grants descritos abaixo; B1 para implementação de código; B3 para autenticação real. P8 para liberar usuários reais.
- **Saídas/artefatos:** página funcional, response envelope C3, migration/constraint necessários, eventos de auditoria, evidências CA.
- **Dependências e responsáveis:** dependências listadas no cabeçalho; Produto/Engenharia fecha B1; Thórus fecha perguntas P referenciadas.
- **Atores e permissões mínimas:** dashboard.read; cada métrica exige read do recurso contado e projeto permitido. Bloco sem permissão fica ausente, sem revelar total global.
- **Superfícies/arquivos/configurações afetadas:** rota, componentes próprios, endpoint/handler, tabelas C2 citadas, autorização C4 e audit event.
- **Risco e plano B:** ocultação visual não protege dados; aplicar a mesma decisão no servidor. Pendência de vocabulário usa somente fixture identificada até resposta P.
- **Rollback ou reversão:** tela pode suspender ação sem apagar histórico; migration aditiva/reversível; cancelamento preserva registro e emite auditoria.

## Página e comportamento detalhado

URL/rota: `/dashboard e /dashboard/{tarefas|alteracoes}`. Base visual única: [Contrato de interface](01-contrato-interface.md).

| Área | Conteúdo e interação |
|---|---|
| Header | Dashboard, “Consultado em [data/hora]”, Atualizar; sem label Sincronizado |
| Cards | Projetos ativos → /projetos; Tarefas vencidas → detalhe tarefas overdue; Próximos prazos → tarefas next7; Alterações pendentes → fila global autorizada |
| Distribuição | Projetos por fase/situação em barras horizontais com texto/quantidade; clicar filtra carteira pelo ID |
| Próximas ações | Cinco tarefas mais próximas (atrasadas primeiro, due asc) com projeto/tarefa/prazo/responsável; Ver todas no detalhe |
| Legal/atividade | Últimos cinco eventos dos 14 dias anteriores incluindo hoje; cada linha abre aba do projeto/registro |
| Detalhe de card | Tabela paginada do objeto contado, com projeto/título/status/prazo, filtros derivados na URL e link ao registro; não apenas projetos sem contexto |

## Dados de entrada e saída

| Métrica | Fórmula |
|---|---|
| Projetos ativos | COUNT projects com archived_at null no scope projects.read |
| Vencidas | COUNT tasks state!=done, due_date<today, projeto ativo, tasks.read |
| Próximos 7 dias | COUNT tasks state!=done, today≤due_date≤today+7; sem data e done excluídas |
| Alterações pendentes | COUNT change_requests state=pending em projetos ativos e definitions.read |
| Fase/situação | Agrupar IDs de projeto ativo; null = Não informado; soma corresponde ao card projetos |
| Legal recente | event_date entre today-13 e today, incluir confirmado/pendente com label; legal_events.read |

## Dados e integrações

UI chama somente API local C3; tabelas respeitam FKs/índices C2. Nenhuma API Asana/Drive é chamada nesta rota.

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Erro |
|---|---|---|---|---|---|
| Tela `/dashboard e /dashboard/{tarefas|alteracoes}` ↔ API local ↔ banco | Banco da aplicação | Campos listados em Dados de entrada; versão/C2 | dashboard.read; cada métrica exige read do recurso contado e projeto permitido. Bloco sem permissão fica ausente, sem revelar total global. | C3: paginação, Idempotency-Key em escrita, expectedVersion em edição; sem retry automático de PATCH | Envelope C3, manter rascunho, correlationId |
| API local ↔ activity_events | Banco na mesma transação | ator, recurso, ação, alvo, instante UTC e resumo seguro | Herda grant da ação | Transação atômica; chave impede repetição de evento | Se log falhar, cancelar mutação e devolver erro C3 |

### Regras de negócio e regras de dados

Snapshot usa mesma data local e política em todas métricas; contador não pode usar filtro diferente do detalhe.
Valor zero ≠ bloco sem permissão; se zero, detalhe vazio específico. Erro mostra “Não foi possível carregar”, não zero.
Mudança de tarefa/pedido invalida cache dos blocos no próximo GET; Atualizar força consulta nova. Sem números fixos no frontend.
P10 usa três projetos representativos e medição humana, não telemetria invasiva adicionada ao MVP.

## API local desta entrega

- GET /api/dashboard → {queriedAt,cards,phaseBuckets,statusBuckets,nextTasks,recentLegal,recentActivity}; aggregates autorizados, snapshot consistente de leitura.
- GET /api/dashboard/tasks?due=overdue|next7&page= → mesmos filtros dos contadores; GET /api/dashboard/change-requests?state=pending&page=.
- Abrir item navega para /projetos/{id}/tarefas?task={taskId}, /definicoes/solicitacoes?request={id}, /legais?event={id}.

## Fluxo principal e caminhos de erro

1. Entrar como CS em A, conferir fixtures overdue/today/+7/+8/done/no-date.
2. Clicar cada card e comparar total de objetos com o número anunciado.
3. Mudar tarefa, atualizar e comparar com usuário que não pode ler tarefas.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Entrar como CS em A, conferir fixtures overdue/today/+7/+8/done/no-date. | Cards e distribuições usam fórmulas/date/permissões definidos, sem projeto B oculto. | Erro de gravação mantém formulário segundo D3. |
| Permissão/limite | dashboard.read; cada métrica exige read do recurso contado e projeto permitido. Bloco sem permissão fica ausente, sem revelar total global. | Detalhes de cada card exibem os mesmos objetos e levam à aba certa. | Responder 401/403/404 conforme C3, sem vazamento. |
| Falha/concorrência | Zero, recurso negado, falha parcial, fronteira +7/+8 e atualização pós-mutação não exibem número enganoso. | Dados permanecem íntegros, sem sucesso falso. | 409/422/503 recuperável; campos mantidos e nenhuma alteração parcial. |

## Critérios de aceite

- [ ] **SPEC-1-015-CA-01:** Cards e distribuições usam fórmulas/date/permissões definidos, sem projeto B oculto.
- [ ] **SPEC-1-015-CA-02:** Detalhes de cada card exibem os mesmos objetos e levam à aba certa.
- [ ] **SPEC-1-015-CA-03:** Zero, recurso negado, falha parcial, fronteira +7/+8 e atualização pós-mutação não exibem número enganoso.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | SPEC-1-015-CA-01 e SPEC-1-015-CA-03 com fixture C5 | Antes da implementação, executar a ação de UI/API descrita nesta SPEC para os critérios principal, negado e conflito | Falha por ausência da regra/comportamento, sem falha de fixture; registrar status/estado observado | Saída do cenário e estado inicial fictício |
| GREEN | Todos os CA desta SPEC | Implementar a rota ponta a ponta; repetir UI/API, reler do banco e tentar request sem grant/com versão obsoleta | Cada CA passa; mutação e evento são atômicos; erro mantém rascunho | Captura por estado, resposta C3 redigida, releitura e evento |
| REFACTOR | Todos os CA desta SPEC | Reexecutar fluxo, teclado e viewports 375/768/1440; aplicar o runner escolhido em B1 quando houver código | Sem regressão; critérios continuam binários; autorização server-side e dados íntegros | Relatório focal e evidências por CA |

**Dados/fixtures:** [Contrato de dados, C5]; adicionar somente as entidades exigidas pela tabela Dados de entrada desta SPEC.  
**Caminhos de erro obrigatórios:** Mudar tarefa, atualizar e comparar com usuário que não pode ler tarefas. Erro de permissão, referência inválida, conflito de versão e falha de banco conforme C3.  
**Evidência exigida:** um artefato por critério; captura de página com dado fictício, API response sem segredo, verificação de persistência/autorização.

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** [contrato visual](01-contrato-interface.md), [dados/API](02-contrato-dados-api.md), seção de limite/permissão desta SPEC e SPEC-1-001, SPEC-1-002, SPEC-1-004, SPEC-1-006, SPEC-1-009, SPEC-1-011, SPEC-1-012, SPEC-1-014.
2. **Alterar somente:** rota `/dashboard e /dashboard/{tarefas|alteracoes}`, componente, handler/endpoint e persistência deste fluxo.
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

- **Como demonstrar:** Mudar tarefa, atualizar e comparar com usuário que não pode ler tarefas.
- **Como operar depois:** Thórus opera pela rota indicada usando os grants C4.
- **Como monitorar:** correlationId, falha de gravação, 401/403, conflito e evento da rota; nenhum log guarda segredo.
- **Pendência conhecida:** B1 para implementação de código; B3 para autenticação real. P8 para liberar usuários reais.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Subseção exata | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| F1-31 | Entregar os indicadores e blocos do Dashboard | Thorus | SPEC-1-015 | Os quatro cards, distribuições e listas exibem valores calculados pelos registros e data de consulta. | Página e comportamento; fórmulas | SPEC-1-015-CA-01; cenário de UI/API, releitura e evidência | Contagem manual comparada a C5 e captura atualizada em. | F1-06/07/11/12/14 e grants read. | Parar se frontend apresentar agregação de universo completo. | Planejada |
| F1-32 | Abrir detalhe exato de cada métrica e verificar fronteiras | Thorus | SPEC-1-015 | Cada card abre os mesmos objetos incluídos na contagem; falha se diferencia de zero. | Fluxo principal e erros; Estados | SPEC-1-015-CA-02; cenário de UI/API, releitura e evidência | Total card vs detalhe; datas hoje/+7/+8; papel sem read. | F1-31, P5/P8/P10. | Parar se card de tarefa levar a lista de projetos sem indicar a tarefa original. | Planejada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
