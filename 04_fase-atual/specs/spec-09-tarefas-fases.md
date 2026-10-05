# SPEC-1-009 — Aba Tarefas e fases: registrar e acompanhar trabalho

**Fase:** 1  
**Status:** planejada  
**Dono:** Engenharia de produto (execução) e Thórus (validação do fluxo)  
**Origem no escopo:** seção 4.1 e 4.9; rota `/projetos/{id}/tarefas`  
**Degrau da solução:** construção mínima da tela e da regra local necessárias para esta operação funcionar manualmente.

## Contexto e decisões fechadas

- **Estado atual:** não existe aplicativo executável neste repositório; a SPEC antiga correspondente está registrada no mapa de migração, commit `a359cdc`.
- **Estado desejado:** uma pessoa com o papel indicado completa a operação em `/projetos/{id}/tarefas` e encontra o resultado persistido, autorizado e auditável.
- **Decisões já fechadas:** Fase 1 é manual; dados próprios ficam no banco C2; auth e grants C3/C4; UI D1–D5; nenhum conector de negócio externo nesta fase.
- **Bloqueios/dependências:** B1 para implementação de código; B3 para autenticação real. P8 para liberar usuários reais. Dependências técnicas de domínio: SPEC-1-001, SPEC-1-002, SPEC-1-004, SPEC-1-005, SPEC-1-008

## Perguntas de validação do cliente

| Perguntas | O que confirmar / como responder | Referência |
|---|---|---|
| P5 e P6: etapas/status e entregas que entram no acompanhamento. | Responder marcando opções ou com uma frase curta. Se não souber, indicar quem confirma. Para Asana, anotar nomes/valores e usar somente GET; não alterar dados nem enviar token. | [Questionário P1–P10](../../03_documentos/06-Questionario-levantamento-cliente.md) |

## Resultado observável

Filtrar vencidas, próximos prazos e sem prazo usando fixtures C5.

## Limites e dependências

- **Inclui:** rota `/projetos/{id}/tarefas`, leitura/mutação desta SPEC, persistência pertinente, grants de servidor, estados D3, activity C3 e evidência abaixo.
- **Fora de escopo:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
- **Entradas e pré-condições:** C2–C5, UI D1–D5, grants descritos abaixo; B1 para implementação de código; B3 para autenticação real. P8 para liberar usuários reais.
- **Saídas/artefatos:** página funcional, response envelope C3, migration/constraint necessários, eventos de auditoria, evidências CA.
- **Dependências e responsáveis:** dependências listadas no cabeçalho; Produto/Engenharia fecha B1; Thórus fecha perguntas P referenciadas.
- **Atores e permissões mínimas:** tasks.read/create/update/complete dentro do projeto; fase local exige projects.update. Responsável é membro ativo.
- **Superfícies/arquivos/configurações afetadas:** rota, componentes próprios, endpoint/handler, tabelas C2 citadas, autorização C4 e audit event.
- **Risco e plano B:** ocultação visual não protege dados; aplicar a mesma decisão no servidor. Pendência de vocabulário usa somente fixture identificada até resposta P.
- **Rollback ou reversão:** tela pode suspender ação sem apagar histórico; migration aditiva/reversível; cancelamento preserva registro e emite auditoria.

## Página e comportamento detalhado

URL/rota: `/projetos/{id}/tarefas`. Base visual única: [Contrato de interface](01-contrato-interface.md).

| Área | Conteúdo e interação |
|---|---|
| Fases | Chips ordenados: Todas / Sem fase / fases do projeto; Gerenciar fases para projects.update |
| Tabela | Título, Fase, Estado, Responsável, Prioridade, Prazo; filtro estado/responsável/prazo e contagem |
| Nova/Editar | Drawer Título, Descrição, Fase, Responsável, Prioridade, Prazo; salvar local |
| Concluir/Reabrir | Ação explícita por linha; concluir altera estado e timestamp; reabrir confirma retorno a Em andamento |
| Indicadores | Vencida em texto se prazo < hoje e não done; Hoje, Próximos 7 dias ou Sem prazo |
| Mobile | Cards com título/estado/prazo/responsável e menu; fase selecionada preservada na URL |

## Dados de entrada e saída

| Campo | Regra |
|---|---|
| title/description | C2; título obrigatório; descrição opcional |
| phase_id/assignee_id/priority | Opcionais; referência existente/ativa e fase pertence ao projeto |
| state | todo=Não iniciada, in_progress=Em andamento, done=Concluída; novo todo; não-configurável |
| due_date | DATE opcional; vencer ontem permitido (projeto já atrasado); completed_at definido servidor |
| phases | kind project_phase, position, active; remover referência desativa fase, sem apagar tarefas |

## Dados e integrações

UI chama somente API local C3; tabelas respeitam FKs/índices C2. Nenhuma API Asana/Drive é chamada nesta rota.

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Erro |
|---|---|---|---|---|---|
| Tela `/projetos/{id}/tarefas` ↔ API local ↔ banco | Banco da aplicação | Campos listados em Dados de entrada; versão/C2 | tasks.read/create/update/complete dentro do projeto; fase local exige projects.update. Responsável é membro ativo. | C3: paginação, Idempotency-Key em escrita, expectedVersion em edição; sem retry automático de PATCH | Envelope C3, manter rascunho, correlationId |
| API local ↔ activity_events | Banco na mesma transação | ator, recurso, ação, alvo, instante UTC e resumo seguro | Herda grant da ação | Transação atômica; chave impede repetição de evento | Se log falhar, cancelar mutação e devolver erro C3 |

### Regras de negócio e regras de dados

Novo todo; todo↔in_progress por editar; todo/in_progress→done somente ação complete; done→in_progress somente reopen.
Prazo: overdue <today; next7 ≥today e ≤today+7; done ou null ficam fora. Date não converte timezone.
Salvar tarefa/auditoria atômicos; lista/dashboard recalculam após commit.
Usuário retirado da equipe permanece legível em tarefa antiga, mas atribuição nova a inativo é recusada.

## API local desta entrega

- GET /api/projects/{id}/tasks?state=&phase=&assignee=&due=overdue|next7|none&page=; sort due asc/null last, depois título/id.
- POST /api/projects/{id}/tasks {title,description,phaseId,assigneeId,priorityOptionId,dueDate}; PATCH /tasks/{taskId} {campos,expectedVersion}.
- POST /tasks/{taskId}/complete ou /reopen {expectedVersion}; done→in_progress ao reabrir.
- PUT /api/projects/{id}/phases {phases,expectedProjectVersion} → configuração local persistida, sem apagar tarefas.

## Fluxo principal e caminhos de erro

1. Selecionar fase, criar tarefa com responsável/prazo e reler lista.
2. Editar estado, concluir e reabrir pela UI.
3. Filtrar vencidas, próximos prazos e sem prazo usando fixtures C5.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Selecionar fase, criar tarefa com responsável/prazo e reler lista. | Tarefa e fase persistem; responsável e filtro usam somente dados do projeto. | Erro de gravação mantém formulário segundo D3. |
| Permissão/limite | tasks.read/create/update/complete dentro do projeto; fase local exige projects.update. Responsável é membro ativo. | Transições atualizam completed_at corretamente e prazo calendário não muda de dia. | Responder 401/403/404 conforme C3, sem vazamento. |
| Falha/concorrência | Projeto arquivado, membro indevido, 409, falha e duplo submit são tratados sem perda ou evento parcial. | Dados permanecem íntegros, sem sucesso falso. | 409/422/503 recuperável; campos mantidos e nenhuma alteração parcial. |

## Critérios de aceite

- [ ] **SPEC-1-009-CA-01:** Tarefa e fase persistem; responsável e filtro usam somente dados do projeto.
- [ ] **SPEC-1-009-CA-02:** Transições atualizam completed_at corretamente e prazo calendário não muda de dia.
- [ ] **SPEC-1-009-CA-03:** Projeto arquivado, membro indevido, 409, falha e duplo submit são tratados sem perda ou evento parcial.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | SPEC-1-009-CA-01 e SPEC-1-009-CA-03 com fixture C5 | Antes da implementação, executar a ação de UI/API descrita nesta SPEC para os critérios principal, negado e conflito | Falha por ausência da regra/comportamento, sem falha de fixture; registrar status/estado observado | Saída do cenário e estado inicial fictício |
| GREEN | Todos os CA desta SPEC | Implementar a rota ponta a ponta; repetir UI/API, reler do banco e tentar request sem grant/com versão obsoleta | Cada CA passa; mutação e evento são atômicos; erro mantém rascunho | Captura por estado, resposta C3 redigida, releitura e evento |
| REFACTOR | Todos os CA desta SPEC | Reexecutar fluxo, teclado e viewports 375/768/1440; aplicar o runner escolhido em B1 quando houver código | Sem regressão; critérios continuam binários; autorização server-side e dados íntegros | Relatório focal e evidências por CA |

**Dados/fixtures:** [Contrato de dados, C5]; adicionar somente as entidades exigidas pela tabela Dados de entrada desta SPEC.  
**Caminhos de erro obrigatórios:** Filtrar vencidas, próximos prazos e sem prazo usando fixtures C5. Erro de permissão, referência inválida, conflito de versão e falha de banco conforme C3.  
**Evidência exigida:** um artefato por critério; captura de página com dado fictício, API response sem segredo, verificação de persistência/autorização.

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** [contrato visual](01-contrato-interface.md), [dados/API](02-contrato-dados-api.md), seção de limite/permissão desta SPEC e SPEC-1-001, SPEC-1-002, SPEC-1-004, SPEC-1-005, SPEC-1-008.
2. **Alterar somente:** rota `/projetos/{id}/tarefas`, componente, handler/endpoint e persistência deste fluxo.
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

- **Como demonstrar:** Filtrar vencidas, próximos prazos e sem prazo usando fixtures C5.
- **Como operar depois:** Thórus opera pela rota indicada usando os grants C4.
- **Como monitorar:** correlationId, falha de gravação, 401/403, conflito e evento da rota; nenhum log guarda segredo.
- **Pendência conhecida:** B1 para implementação de código; B3 para autenticação real. P8 para liberar usuários reais.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Subseção exata | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| F1-07 | Implementar tarefas, fases e mudanças de estado na aba do projeto | Thorus | SPEC-1-009 | Usuário permitido cria/edita/conclui/reabre tarefa e relê sua data/estado persistidos. | Página e comportamento detalhado; Transições e regras | SPEC-1-009-CA-01; cenário de UI/API, releitura e evidência | Roteiro por estado e prazo C5; registro de leitura após nova sessão. | P5 confirma listas; SPEC-1-005/008 e C2/C4. | Parar se fase pertencer a outro projeto ou done não preservar timestamp. | Planejada |
| F1-25 | Implementar transições, prazo e filtros da aba de tarefas | Thorus | SPEC-1-009 | Tarefa concluída/removida do vencido; janela próximos 7 dias inclui +7 e exclui +8. | Transições e regras; Estados e erros | SPEC-1-009-CA-02; cenário de UI/API, releitura e evidência | Tabela com fixtures C5, URL filtros e data DATE estável. | F1-07, P5 e SPEC-1-005. | Parar se data virar outro dia por conversão de fuso ou reabrir duplicar histórico. | Planejada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
