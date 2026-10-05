# SPEC-1-012 — Aba Legais e marcos com evidências e confirmação

**Fase:** 1  
**Status:** planejada  
**Dono:** Engenharia de produto (execução) e Thórus (validação do fluxo)  
**Origem no escopo:** seção 4.6 e 4.9; rota `/projetos/{id}/legais`  
**Degrau da solução:** construção mínima da tela e da regra local necessárias para esta operação funcionar manualmente.

## Contexto e decisões fechadas

- **Estado atual:** não existe aplicativo executável neste repositório; a SPEC antiga correspondente está registrada no mapa de migração, commit `a359cdc`.
- **Estado desejado:** uma pessoa com o papel indicado completa a operação em `/projetos/{id}/legais` e encontra o resultado persistido, autorizado e auditável.
- **Decisões já fechadas:** Fase 1 é manual; dados próprios ficam no banco C2; auth e grants C3/C4; UI D1–D5; nenhum conector de negócio externo nesta fase.
- **Bloqueios/dependências:** P5 deve classificar tipos previstos/fatos e exigir evidência; antes disso só demonstração com exemplos claramente fictícios. Dependências técnicas de domínio: SPEC-1-001, SPEC-1-002, SPEC-1-004, SPEC-1-005, SPEC-1-008

## Perguntas de validação do cliente

| Perguntas | O que confirmar / como responder | Referência |
|---|---|---|
| P5–P6: marcos legais, responsáveis, comprovação e frequência de comunicação. | Responder marcando opções ou com uma frase curta. Se não souber, indicar quem confirma. Para Asana, anotar nomes/valores e usar somente GET; não alterar dados nem enviar token. | [Questionário P1–P10](../../03_documentos/06-Questionario-levantamento-cliente.md) |

## Resultado observável

Editar dado confirmado exige nova confirmação e mostra histórico de correção.

## Limites e dependências

- **Inclui:** rota `/projetos/{id}/legais`, leitura/mutação desta SPEC, persistência pertinente, grants de servidor, estados D3, activity C3 e evidência abaixo.
- **Fora de escopo:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
- **Entradas e pré-condições:** C2–C5, UI D1–D5, grants descritos abaixo; P5 deve classificar tipos previstos/fatos e exigir evidência; antes disso só demonstração com exemplos claramente fictícios.
- **Saídas/artefatos:** página funcional, response envelope C3, migration/constraint necessários, eventos de auditoria, evidências CA.
- **Dependências e responsáveis:** dependências listadas no cabeçalho; Produto/Engenharia fecha B1; Thórus fecha perguntas P referenciadas.
- **Atores e permissões mínimas:** legal_events.read/create/update para registro; confirmar fato legal exige legal_events.confirm. Tipos/status reais vêm de P5.
- **Superfícies/arquivos/configurações afetadas:** rota, componentes próprios, endpoint/handler, tabelas C2 citadas, autorização C4 e audit event.
- **Risco e plano B:** ocultação visual não protege dados; aplicar a mesma decisão no servidor. Pendência de vocabulário usa somente fixture identificada até resposta P.
- **Rollback ou reversão:** tela pode suspender ação sem apagar histórico; migration aditiva/reversível; cancelamento preserva registro e emite auditoria.

## Página e comportamento detalhado

URL/rota: `/projetos/{id}/legais`. Base visual única: [Contrato de interface](01-contrato-interface.md).

| Área | Conteúdo e interação |
|---|---|
| Lista | Tipo, Status, Data, Responsável, Protocolo, Confirmado/Pendente; filtro tipo/status/período |
| Registrar/editar | Drawer Tipo, Status, Data, Responsável, Protocolo, Observação, Link de evidência |
| Confirmar | Resumo do fato + evidência; CTA Confirmar registro; indicador autor/hora |
| Histórico | Link Ver atividade para o registro; correções de dado confirmado voltam a Pendente |
| Avisos | “Registro interno; nenhuma comunicação enviada”; URL é referência não verificação legal |

## Dados de entrada e saída

| Campo | Regra |
|---|---|
| type/status/date/responsible | Obrigatórios C2; listas ativas; responsável membro ativo; calendário inclusive passado |
| protocol/note | Opcionais, limites C2 |
| evidenceUrl | HTTP(S); exigência por tipo definida em P5; piloto considera evidência obrigatória para confirmar aprovação/parecer |
| confirmedBy/At | Servidor; editar fato confirmado limpa confirmação e audita correção |

## Dados e integrações

UI chama somente API local C3; tabelas respeitam FKs/índices C2. Nenhuma API Asana/Drive é chamada nesta rota.

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Erro |
|---|---|---|---|---|---|
| Tela `/projetos/{id}/legais` ↔ API local ↔ banco | Banco da aplicação | Campos listados em Dados de entrada; versão/C2 | legal_events.read/create/update para registro; confirmar fato legal exige legal_events.confirm. Tipos/status reais vêm de P5. | C3: paginação, Idempotency-Key em escrita, expectedVersion em edição; sem retry automático de PATCH | Envelope C3, manter rascunho, correlationId |
| API local ↔ activity_events | Banco na mesma transação | ator, recurso, ação, alvo, instante UTC e resumo seguro | Herda grant da ação | Transação atômica; chave impede repetição de evento | Se log falhar, cancelar mutação e devolver erro C3 |

### Regras de negócio e regras de dados

Catálogo legal é distinto de estado técnico/tarefa/saúde geral; não reutilizar valores por coincidência de label.
Confirmar valida completude/evidência por tipo e registra fato humano; não certificar engenharia nem enviar mensagem.
Correção conserva evento anterior na auditoria e invalida confirmação; autor escolhe confirmar novamente.
Data futura pode registrar marco previsto, mas confirmar parecer/aprovação exige data de fato não futura; P5 define tipo previsto versus fato.

## API local desta entrega

- GET /api/projects/{id}/legal-events?type=&status=&from=&to=&confirmed=&page= → date desc/id.
- POST /api/projects/{id}/legal-events {typeId,statusId,eventDate,responsibleId,protocol,note,evidenceUrl}; PATCH /legal-events/{id} {campos,expectedVersion}.
- POST /legal-events/{id}/confirm {expectedVersion} → exige permissão/regra de evidência, grava autor/time.

## Fluxo principal e caminhos de erro

1. Legais registra protocolo ou marco previsto com data/responsável.
2. Registra parecer com URL e papel autorizado confirma.
3. Editar dado confirmado exige nova confirmação e mostra histórico de correção.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Legais registra protocolo ou marco previsto com data/responsável. | Evento completo persiste e filtros retornam seu tipo/status/data corretos. | Erro de gravação mantém formulário segundo D3. |
| Permissão/limite | legal_events.read/create/update para registro; confirmar fato legal exige legal_events.confirm. Tipos/status reais vêm de P5. | Confirmação exige autorização/evidência; correção invalida e mantém autoria anterior. | Responder 401/403/404 conforme C3, sem vazamento. |
| Falha/concorrência | Tipo inativo, falta de evidência, data futura de fato e falha de gravação exibem erros específicos. | Dados permanecem íntegros, sem sucesso falso. | 409/422/503 recuperável; campos mantidos e nenhuma alteração parcial. |

## Critérios de aceite

- [ ] **SPEC-1-012-CA-01:** Evento completo persiste e filtros retornam seu tipo/status/data corretos.
- [ ] **SPEC-1-012-CA-02:** Confirmação exige autorização/evidência; correção invalida e mantém autoria anterior.
- [ ] **SPEC-1-012-CA-03:** Tipo inativo, falta de evidência, data futura de fato e falha de gravação exibem erros específicos.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | SPEC-1-012-CA-01 e SPEC-1-012-CA-03 com fixture C5 | Antes da implementação, executar a ação de UI/API descrita nesta SPEC para os critérios principal, negado e conflito | Falha por ausência da regra/comportamento, sem falha de fixture; registrar status/estado observado | Saída do cenário e estado inicial fictício |
| GREEN | Todos os CA desta SPEC | Implementar a rota ponta a ponta; repetir UI/API, reler do banco e tentar request sem grant/com versão obsoleta | Cada CA passa; mutação e evento são atômicos; erro mantém rascunho | Captura por estado, resposta C3 redigida, releitura e evento |
| REFACTOR | Todos os CA desta SPEC | Reexecutar fluxo, teclado e viewports 375/768/1440; aplicar o runner escolhido em B1 quando houver código | Sem regressão; critérios continuam binários; autorização server-side e dados íntegros | Relatório focal e evidências por CA |

**Dados/fixtures:** [Contrato de dados, C5]; adicionar somente as entidades exigidas pela tabela Dados de entrada desta SPEC.  
**Caminhos de erro obrigatórios:** Editar dado confirmado exige nova confirmação e mostra histórico de correção. Erro de permissão, referência inválida, conflito de versão e falha de banco conforme C3.  
**Evidência exigida:** um artefato por critério; captura de página com dado fictício, API response sem segredo, verificação de persistência/autorização.

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** [contrato visual](01-contrato-interface.md), [dados/API](02-contrato-dados-api.md), seção de limite/permissão desta SPEC e SPEC-1-001, SPEC-1-002, SPEC-1-004, SPEC-1-005, SPEC-1-008.
2. **Alterar somente:** rota `/projetos/{id}/legais`, componente, handler/endpoint e persistência deste fluxo.
3. **Não alterar:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
4. **Executar nesta ordem:** migration/contrato; leitura autorizada; ação principal e commit; estado de erro; teclado/mobile; captura de evidência.
5. **Parar e pedir validação quando:** P5 deve classificar tipos previstos/fatos e exigir evidência; antes disso só demonstração com exemplos claramente fictícios.
6. **Estado válido ao parar:** rota principal não grava parcial; sessão/permissões protegem resposta; dados existentes e histórico continuam íntegros.

## Checklist de execução

- [ ] Grant e lista de campos confirmados; fixture mínima tem chave/teste repetível.
- [ ] Caminho principal liga tela, servidor, banco e evento de auditoria na mesma entrega.
- [ ] Vazio, erro, inválido, negado, concorrência e sucesso aplicam D3 na rota.
- [ ] Teclado/foco/viewport e evidências por CA verificados.
- [ ] Thórus confere o fluxo com dados fictícios; pendência de produto registrada.

## Handoff e operação

- **Como demonstrar:** Editar dado confirmado exige nova confirmação e mostra histórico de correção.
- **Como operar depois:** Thórus opera pela rota indicada usando os grants C4.
- **Como monitorar:** correlationId, falha de gravação, 401/403, conflito e evento da rota; nenhum log guarda segredo.
- **Pendência conhecida:** P5 deve classificar tipos previstos/fatos e exigir evidência; antes disso só demonstração com exemplos claramente fictícios.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Subseção exata | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| F1-12 | Implementar eventos legais com confirmação e evidência | Thorus | SPEC-1-012 | Evento permitido persiste; confirmar registra pessoa/data e exige campos/evidência definidos em P5. | Página e comportamento detalhado; Regras de negócio | SPEC-1-012-CA-01; cenário de UI/API, releitura e evidência | Capturas do evento pendente/confirmado e correção com auditoria. | P5 define fato versus previsão e exigência de evidência. | Parar se estado/tipo/autorizador do evento real não estiver decidido. | Planejada |
| F1-28 | Implementar correção, confirmação e auditabilidade de evento legal | Thorus | SPEC-1-012 | Correção de evento confirmado revoga confirmação e conserva seu estado anterior auditado. | API local; Regras de negócio | SPEC-1-012-CA-02; cenário de UI/API, releitura e evidência | Comparação antes/depois e tentativa sem permission. | F1-12, F1-08 e P5. | Parar se ausência de prova permitir afirmar aprovação legal. | Planejada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
