# SPEC-1-004 — Administração de funções e níveis de permissão

**Fase:** 1  
**Status:** planejada  
**Dono:** Engenharia de produto (execução) e Thórus (validação do fluxo)  
**Origem no escopo:** seção 3 e 4.9; rota `/configuracoes/perfis`  
**Degrau da solução:** construção mínima da tela e da regra local necessárias para esta operação funcionar manualmente.

## Contexto e decisões fechadas

- **Estado atual:** não existe aplicativo executável neste repositório; a SPEC antiga correspondente está registrada no mapa de migração, commit `a359cdc`.
- **Estado desejado:** uma pessoa com o papel indicado completa a operação em `/configuracoes/perfis` e encontra o resultado persistido, autorizado e auditável.
- **Decisões já fechadas:** Fase 1 é manual; dados próprios ficam no banco C2; auth e grants C3/C4; UI D1–D5; nenhum conector de negócio externo nesta fase.
- **Bloqueios/dependências:** P8/P9 confirmam política real. Demonstração usa a base proposta C4 e dois projetos fictícios. Dependências técnicas de domínio: SPEC-1-001, SPEC-1-002

## Perguntas de validação do cliente

| Perguntas | O que confirmar / como responder | Referência |
|---|---|---|
| P8: para cada perfil, marcar projetos próprios/todos e documentos liberados; confirmar grants recurso × ação × escopo. | Responder marcando opções ou com uma frase curta. Se não souber, indicar quem confirma. Para Asana, anotar nomes/valores e usar somente GET; não alterar dados nem enviar token. | [Questionário P1–P10](../../03_documentos/06-Questionario-levantamento-cliente.md) |

## Resultado observável

CS tenta concluir tarefa pela UI e por chamada direta; ambas negam na requisição seguinte.

## Limites e dependências

- **Inclui:** rota `/configuracoes/perfis`, leitura/mutação desta SPEC, persistência pertinente, grants de servidor, estados D3, activity C3 e evidência abaixo.
- **Fora de escopo:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
- **Entradas e pré-condições:** C2–C5, UI D1–D5, grants descritos abaixo; P8/P9 confirmam política real. Demonstração usa a base proposta C4 e dois projetos fictícios.
- **Saídas/artefatos:** página funcional, response envelope C3, migration/constraint necessários, eventos de auditoria, evidências CA.
- **Dependências e responsáveis:** dependências listadas no cabeçalho; Produto/Engenharia fecha B1; Thórus fecha perguntas P referenciadas.
- **Atores e permissões mínimas:** Somente Admin com admin_profiles.read/update; matriz controla recursos/ações C4. Aprovação técnica também exige designação projeto/disciplina.
- **Superfícies/arquivos/configurações afetadas:** rota, componentes próprios, endpoint/handler, tabelas C2 citadas, autorização C4 e audit event.
- **Risco e plano B:** ocultação visual não protege dados; aplicar a mesma decisão no servidor. Pendência de vocabulário usa somente fixture identificada até resposta P.
- **Rollback ou reversão:** tela pode suspender ação sem apagar histórico; migration aditiva/reversível; cancelamento preserva registro e emite auditoria.

## Página e comportamento detalhado

URL/rota: `/configuracoes/perfis`. Base visual única: [Contrato de interface](01-contrato-interface.md).

| Área | Conteúdo e interação |
|---|---|
| Perfis | Seleção dos cinco perfis fixos com descrição e quantidade de usuários |
| Matriz | Linhas por recurso/função C4; colunas leitura/criação/edição/conclusão/arquivamento/aprovação quando aplicável; checkbox com nome completo |
| Nível de acesso | Select “Projetos atribuídos” / “Todos os projetos” por recurso; sem opção todos se o usuário não tiver read |
| Resumo de alterações | Antes/depois, permissões concedidas/retiradas e total de usuários afetados; “Revisar e salvar” |
| Restrições | Células administrativas fixas para Admin com motivo; liderança base somente leitura, editável após P8; “Restaurar base do piloto” apenas preenche rascunho, ainda exige salvar |

## Dados de entrada e saída

| Campo | Regra |
|---|---|
| role_id | Um perfil por operação; nomes-chave fixos |
| permissions[] | Catálogo resource/action C4; scope assigned/all; ações inexistentes rejeitadas |
| expectedPolicyVersion | Obrigatório; two admins with stale snapshot → 409 |
| reason | Motivo obrigatório 3–500, mostrado na auditoria administrativa |
| preview | Cálculo no servidor dos usuários afetados e validação dos grants, sem gravar |

## Dados e integrações

UI chama somente API local C3; tabelas respeitam FKs/índices C2. Nenhuma API Asana/Drive é chamada nesta rota.

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Erro |
|---|---|---|---|---|---|
| Tela `/configuracoes/perfis` ↔ API local ↔ banco | Banco da aplicação | Campos listados em Dados de entrada; versão/C2 | Somente Admin com admin_profiles.read/update; matriz controla recursos/ações C4. Aprovação técnica também exige designação projeto/disciplina. | C3: paginação, Idempotency-Key em escrita, expectedVersion em edição; sem retry automático de PATCH | Envelope C3, manter rascunho, correlationId |
| API local ↔ activity_events | Banco na mesma transação | ator, recurso, ação, alvo, instante UTC e resumo seguro | Herda grant da ação | Transação atômica; chave impede repetição de evento | Se log falhar, cancelar mutação e devolver erro C3 |

### Regras de negócio e regras de dados

Política C4 governa matriz e servidor; ausência de grant nega, write exige read e projects.read.
Um único perfil por usuário elimina soma/interseção implícita de permissões; concessões de Admin não podem remover gestão mínima.
Alterar scope assigned→all expande visibilidade; modal explicita mudança antes do commit.
Duas edições concorrentes não se sobrescrevem. Confirmação de versão antiga exige recarregar e revisar.
Definitions.decide não basta: project_approvers por disciplina é verificado na aprovação.

## API local desta entrega

- GET /api/admin/roles e GET /api/admin/roles/{id}/permissions → matriz/version.
- POST /api/admin/roles/{id}/permissions/preview {permissions,expectedPolicyVersion,reason} → delta e quantidade de usuários afetados.
- PUT /api/admin/roles/{id}/permissions {permissions,expectedPolicyVersion,reason} → transação substitui grants, incrementa policyVersion e gera auditoria.

## Fluxo principal e caminhos de erro

1. Admin seleciona CS, desmarca tasks.complete e mantém leitura/criação.
2. Revisar mostra função retirada e número de afetados; Salvar atualiza versão.
3. CS tenta concluir tarefa pela UI e por chamada direta; ambas negam na requisição seguinte.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Admin seleciona CS, desmarca tasks.complete e mantém leitura/criação. | Admin edita por função/escopo e preview representa o delta que será persistido. | Erro de gravação mantém formulário segundo D3. |
| Permissão/limite | Somente Admin com admin_profiles.read/update; matriz controla recursos/ações C4. Aprovação técnica também exige designação projeto/disciplina. | Permissão retirada vale na próxima requisição, incluindo endpoint direto e métricas. | Responder 401/403/404 conforme C3, sem vazamento. |
| Falha/concorrência | PolicyVersion obsoleta, write sem read, expansão all e proteção de Admin são tratados sem sobrescrita. | Dados permanecem íntegros, sem sucesso falso. | 409/422/503 recuperável; campos mantidos e nenhuma alteração parcial. |

## Critérios de aceite

- [ ] **SPEC-1-004-CA-01:** Admin edita por função/escopo e preview representa o delta que será persistido.
- [ ] **SPEC-1-004-CA-02:** Permissão retirada vale na próxima requisição, incluindo endpoint direto e métricas.
- [ ] **SPEC-1-004-CA-03:** PolicyVersion obsoleta, write sem read, expansão all e proteção de Admin são tratados sem sobrescrita.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | SPEC-1-004-CA-01 e SPEC-1-004-CA-03 com fixture C5 | Antes da implementação, executar a ação de UI/API descrita nesta SPEC para os critérios principal, negado e conflito | Falha por ausência da regra/comportamento, sem falha de fixture; registrar status/estado observado | Saída do cenário e estado inicial fictício |
| GREEN | Todos os CA desta SPEC | Implementar a rota ponta a ponta; repetir UI/API, reler do banco e tentar request sem grant/com versão obsoleta | Cada CA passa; mutação e evento são atômicos; erro mantém rascunho | Captura por estado, resposta C3 redigida, releitura e evento |
| REFACTOR | Todos os CA desta SPEC | Reexecutar fluxo, teclado e viewports 375/768/1440; aplicar o runner escolhido em B1 quando houver código | Sem regressão; critérios continuam binários; autorização server-side e dados íntegros | Relatório focal e evidências por CA |

**Dados/fixtures:** [Contrato de dados, C5]; adicionar somente as entidades exigidas pela tabela Dados de entrada desta SPEC.  
**Caminhos de erro obrigatórios:** CS tenta concluir tarefa pela UI e por chamada direta; ambas negam na requisição seguinte. Erro de permissão, referência inválida, conflito de versão e falha de banco conforme C3.  
**Evidência exigida:** um artefato por critério; captura de página com dado fictício, API response sem segredo, verificação de persistência/autorização.

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** [contrato visual](01-contrato-interface.md), [dados/API](02-contrato-dados-api.md), seção de limite/permissão desta SPEC e SPEC-1-001, SPEC-1-002.
2. **Alterar somente:** rota `/configuracoes/perfis`, componente, handler/endpoint e persistência deste fluxo.
3. **Não alterar:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
4. **Executar nesta ordem:** migration/contrato; leitura autorizada; ação principal e commit; estado de erro; teclado/mobile; captura de evidência.
5. **Parar e pedir validação quando:** P8/P9 confirmam política real. Demonstração usa a base proposta C4 e dois projetos fictícios.
6. **Estado válido ao parar:** rota principal não grava parcial; sessão/permissões protegem resposta; dados existentes e histórico continuam íntegros.

## Checklist de execução

- [ ] Grant e lista de campos confirmados; fixture mínima tem chave/teste repetível.
- [ ] Caminho principal liga tela, servidor, banco e evento de auditoria na mesma entrega.
- [ ] Vazio, erro, inválido, negado, concorrência e sucesso aplicam D3 na rota.
- [ ] Teclado/foco/viewport e evidências por CA verificados.
- [ ] Thórus confere o fluxo com dados fictícios; pendência de produto registrada.

## Handoff e operação

- **Como demonstrar:** CS tenta concluir tarefa pela UI e por chamada direta; ambas negam na requisição seguinte.
- **Como operar depois:** Thórus opera pela rota indicada usando os grants C4.
- **Como monitorar:** correlationId, falha de gravação, 401/403, conflito e evento da rota; nenhum log guarda segredo.
- **Pendência conhecida:** P8/P9 confirmam política real. Demonstração usa a base proposta C4 e dois projetos fictícios.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Subseção exata | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| F1-02 | Aprovar os cinco perfis e o escopo atribuído ou transversal | Thorus | SPEC-1-004 | Thórus confirma grants e escopos da matriz C4 por recurso para Admin, CS, Engenharia, Legais e Liderança. | Permissões e decisões fechadas | SPEC-1-004-CA-01; cenário de UI/API, releitura e evidência | Matriz papel × ação × escopo assinada e casos permitido/negado definidos. | P8 respondida; B1 identifica limite de provisionamento de role. | Parar antes de habilitar usuário real se leitura, escrita ou escopo all estiver em disputa. | Planejada |
| F1-19 | Editar matriz Admin, revisar impacto e aplicar grants em servidor | Thorus | SPEC-1-004 | Preview e salvar produzem o mesmo delta; próximo request aplica policyVersion nova. | Página e comportamento; API local | SPEC-1-004-CA-02; cenário de UI/API, releitura e evidência | Matriz antes/depois, policy_version, endpoint permitido e negado. | F1-02, P8 e runtime B1. | Parar em grant sem read, ampliação all sem confirmação ou Admin sem recuperação. | Planejada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
