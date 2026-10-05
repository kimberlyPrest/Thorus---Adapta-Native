# SPEC-1-011 — Solicitar, revisar e decidir alteração técnica

**Fase:** 1  
**Status:** planejada  
**Dono:** Engenharia de produto (execução) e Thórus (validação do fluxo)  
**Origem no escopo:** seção 4.4 e 4.9; rota `/projetos/{id}/definicoes/solicitacoes`  
**Degrau da solução:** construção mínima da tela e da regra local necessárias para esta operação funcionar manualmente.

## Contexto e decisões fechadas

- **Estado atual:** não existe aplicativo executável neste repositório; a SPEC antiga correspondente está registrada no mapa de migração, commit `a359cdc`.
- **Estado desejado:** uma pessoa com o papel indicado completa a operação em `/projetos/{id}/definicoes/solicitacoes` e encontra o resultado persistido, autorizado e auditável.
- **Decisões já fechadas:** Fase 1 é manual; dados próprios ficam no banco C2; auth e grants C3/C4; UI D1–D5; nenhum conector de negócio externo nesta fase.
- **Bloqueios/dependências:** P9 fecha autoridade e regra de solicitante/aprovador para fatos reais. Nunca completar aprovação por inferência. Dependências técnicas de domínio: SPEC-1-001, SPEC-1-002, SPEC-1-004, SPEC-1-008, SPEC-1-010

## Perguntas de validação do cliente

| Perguntas | O que confirmar / como responder | Referência |
|---|---|---|
| P8–P9: aprovadores, impacto, aprovação e quando a definição oficial muda. | Responder marcando opções ou com uma frase curta. Se não souber, indicar quem confirma. Para Asana, anotar nomes/valores e usar somente GET; não alterar dados nem enviar token. | [Questionário P1–P10](../../03_documentos/06-Questionario-levantamento-cliente.md) |

## Resultado observável

Após aprovar, voltar às Vigentes mostra v2 e histórico v1; repetir decisão/reenvio não duplica versão.

## Limites e dependências

- **Inclui:** rota `/projetos/{id}/definicoes/solicitacoes`, leitura/mutação desta SPEC, persistência pertinente, grants de servidor, estados D3, activity C3 e evidência abaixo.
- **Fora de escopo:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
- **Entradas e pré-condições:** C2–C5, UI D1–D5, grants descritos abaixo; P9 fecha autoridade e regra de solicitante/aprovador para fatos reais. Nunca completar aprovação por inferência.
- **Saídas/artefatos:** página funcional, response envelope C3, migration/constraint necessários, eventos de auditoria, evidências CA.
- **Dependências e responsáveis:** dependências listadas no cabeçalho; Produto/Engenharia fecha B1; Thórus fecha perguntas P referenciadas.
- **Atores e permissões mínimas:** definitions.propose cria pedido; definitions.decide + aprovador projeto/disciplina decide. Autor pode editar/cancelar seu rascunho ou cancelar pedido pendente; não autoaprova por ser autor.
- **Superfícies/arquivos/configurações afetadas:** rota, componentes próprios, endpoint/handler, tabelas C2 citadas, autorização C4 e audit event.
- **Risco e plano B:** ocultação visual não protege dados; aplicar a mesma decisão no servidor. Pendência de vocabulário usa somente fixture identificada até resposta P.
- **Rollback ou reversão:** tela pode suspender ação sem apagar histórico; migration aditiva/reversível; cancelamento preserva registro e emite auditoria.

## Página e comportamento detalhado

URL/rota: `/projetos/{id}/definicoes/solicitacoes`. Base visual única: [Contrato de interface](01-contrato-interface.md).

| Área | Conteúdo e interação |
|---|---|
| Fila | Status, disciplina, definição, solicitante, data; filtros pending/draft/decididos |
| Proposta | Drawer Valor atual (readonly), Proposto, Motivo, Impacto informado, Origem, URL opcional; Salvar rascunho / Enviar para revisão |
| Revisão | Página/drawer com valor-base, vigente atual, proposto e metadados; Aprovador e autoridade legíveis |
| Decisão | Aprovar/Recusar com justificativa; confirmar efeito técnico; aprovar mostra próximo número de versão |
| Base desatualizada | Banner de conflito; comparar três valores; solicitante revisa/reenquadra pedido com base atual e reenvia |
| Histórico | Pedidos decididos só leitura; status em texto, decisão/data/autor visíveis |

## Dados de entrada e saída

| Campo | Regra |
|---|---|
| definitionId/baseVersionId | Obrigatórios; base copiada da vigente ao abrir formulário |
| proposedValue/reason | Obrigatórios para enviar, limites C2; rascunho pode ficar incompleto |
| impactNote/origin/sourceUrl | Opcionais; origin enum C2; URL manual não importa texto |
| decisionNote | 3–2000 obrigatório aprovar/recusar; decidedBy/time vem do servidor |
| expectedVersion | Da solicitação; aprovação também verifica vigente atual = baseVersionId |

## Dados e integrações

UI chama somente API local C3; tabelas respeitam FKs/índices C2. Nenhuma API Asana/Drive é chamada nesta rota.

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Erro |
|---|---|---|---|---|---|
| Tela `/projetos/{id}/definicoes/solicitacoes` ↔ API local ↔ banco | Banco da aplicação | Campos listados em Dados de entrada; versão/C2 | definitions.propose cria pedido; definitions.decide + aprovador projeto/disciplina decide. Autor pode editar/cancelar seu rascunho ou cancelar pedido pendente; não autoaprova por ser autor. | C3: paginação, Idempotency-Key em escrita, expectedVersion em edição; sem retry automático de PATCH | Envelope C3, manter rascunho, correlationId |
| API local ↔ activity_events | Banco na mesma transação | ator, recurso, ação, alvo, instante UTC e resumo seguro | Herda grant da ação | Transação atômica; chave impede repetição de evento | Se log falhar, cancelar mutação e devolver erro C3 |

### Regras de negócio e regras de dados

Transições: draft→pending (autor com campos completos); draft/pending→cancelled (autor); pending→draft (autor para revisar); pending→approved/rejected (aprovador elegível). Estados finais não reabrem; novo pedido referencia a vigente.
Duas aprovações concorrentes de pedidos base v1: primeira gera v2; segunda 409 base desatualizada, sem criar v3 silenciosa.
Mesmo aprovador/solicitante só decide se designado P9; não presumir separação obrigatória de pessoas, registrar política humana em P9 antes do piloto real.
Recusar/cancelar mantém vigente; aprovação substitui ponteiro mas preserva todas versões. Não gerar aviso externo ou efeito de preço/prazo automaticamente.

## API local desta entrega

- POST /definitions/{id}/requests {baseVersionId,proposedValue,reason,impactNote,origin,sourceUrl,state:draft}; GET /api/projects/{id}/change-requests?state=&discipline=.
- PATCH /change-requests/{id} {proposedValue,reason,impactNote,sourceUrl,baseVersionId,expectedVersion} somente draft/autor; revisão de pending requer ação /return-to-draft do autor.
- POST /change-requests/{id}/submit,/cancel,/return-to-draft {expectedVersion}; POST /approve,/reject {expectedVersion,decisionNote}.
- Approve faz lock/check de current_version e versão do pedido, gera vN+1, atualiza ponteiro, state approved e auditoria atômicos.

## Fluxo principal e caminhos de erro

1. Engenharia cria rascunho v1→novo valor e envia para revisão.
2. Aprovador compara valor-base/atual/proposto e confirma com justificativa.
3. Após aprovar, voltar às Vigentes mostra v2 e histórico v1; repetir decisão/reenvio não duplica versão.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Engenharia cria rascunho v1→novo valor e envia para revisão. | Rascunho e envio mantêm versão-base, campos e autoria; vigente não muda no submit. | Erro de gravação mantém formulário segundo D3. |
| Permissão/limite | definitions.propose cria pedido; definitions.decide + aprovador projeto/disciplina decide. Autor pode editar/cancelar seu rascunho ou cancelar pedido pendente; não autoaprova por ser autor. | Aprovar/rejeitar seguem transições/designação e transação; aprovando gera uma única vigente nova. | Responder 401/403/404 conforme C3, sem vazamento. |
| Falha/concorrência | Proposta obsoleta, decisão dupla, papel indevido e falha conservam vigente e histórico sem versão extra. | Dados permanecem íntegros, sem sucesso falso. | 409/422/503 recuperável; campos mantidos e nenhuma alteração parcial. |

## Critérios de aceite

- [ ] **SPEC-1-011-CA-01:** Rascunho e envio mantêm versão-base, campos e autoria; vigente não muda no submit.
- [ ] **SPEC-1-011-CA-02:** Aprovar/rejeitar seguem transições/designação e transação; aprovando gera uma única vigente nova.
- [ ] **SPEC-1-011-CA-03:** Proposta obsoleta, decisão dupla, papel indevido e falha conservam vigente e histórico sem versão extra.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | SPEC-1-011-CA-01 e SPEC-1-011-CA-03 com fixture C5 | Antes da implementação, executar a ação de UI/API descrita nesta SPEC para os critérios principal, negado e conflito | Falha por ausência da regra/comportamento, sem falha de fixture; registrar status/estado observado | Saída do cenário e estado inicial fictício |
| GREEN | Todos os CA desta SPEC | Implementar a rota ponta a ponta; repetir UI/API, reler do banco e tentar request sem grant/com versão obsoleta | Cada CA passa; mutação e evento são atômicos; erro mantém rascunho | Captura por estado, resposta C3 redigida, releitura e evento |
| REFACTOR | Todos os CA desta SPEC | Reexecutar fluxo, teclado e viewports 375/768/1440; aplicar o runner escolhido em B1 quando houver código | Sem regressão; critérios continuam binários; autorização server-side e dados íntegros | Relatório focal e evidências por CA |

**Dados/fixtures:** [Contrato de dados, C5]; adicionar somente as entidades exigidas pela tabela Dados de entrada desta SPEC.  
**Caminhos de erro obrigatórios:** Após aprovar, voltar às Vigentes mostra v2 e histórico v1; repetir decisão/reenvio não duplica versão. Erro de permissão, referência inválida, conflito de versão e falha de banco conforme C3.  
**Evidência exigida:** um artefato por critério; captura de página com dado fictício, API response sem segredo, verificação de persistência/autorização.

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** [contrato visual](01-contrato-interface.md), [dados/API](02-contrato-dados-api.md), seção de limite/permissão desta SPEC e SPEC-1-001, SPEC-1-002, SPEC-1-004, SPEC-1-008, SPEC-1-010.
2. **Alterar somente:** rota `/projetos/{id}/definicoes/solicitacoes`, componente, handler/endpoint e persistência deste fluxo.
3. **Não alterar:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
4. **Executar nesta ordem:** migration/contrato; leitura autorizada; ação principal e commit; estado de erro; teclado/mobile; captura de evidência.
5. **Parar e pedir validação quando:** P9 fecha autoridade e regra de solicitante/aprovador para fatos reais. Nunca completar aprovação por inferência.
6. **Estado válido ao parar:** rota principal não grava parcial; sessão/permissões protegem resposta; dados existentes e histórico continuam íntegros.

## Checklist de execução

- [ ] Grant e lista de campos confirmados; fixture mínima tem chave/teste repetível.
- [ ] Caminho principal liga tela, servidor, banco e evento de auditoria na mesma entrega.
- [ ] Vazio, erro, inválido, negado, concorrência e sucesso aplicam D3 na rota.
- [ ] Teclado/foco/viewport e evidências por CA verificados.
- [ ] Thórus confere o fluxo com dados fictícios; pendência de produto registrada.

## Handoff e operação

- **Como demonstrar:** Após aprovar, voltar às Vigentes mostra v2 e histórico v1; repetir decisão/reenvio não duplica versão.
- **Como operar depois:** Thórus opera pela rota indicada usando os grants C4.
- **Como monitorar:** correlationId, falha de gravação, 401/403, conflito e evento da rota; nenhum log guarda segredo.
- **Pendência conhecida:** P9 fecha autoridade e regra de solicitante/aprovador para fatos reais. Nunca completar aprovação por inferência.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Subseção exata | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| F1-11 | Implementar pedidos e decisão técnica com versão imutável | Thorus | SPEC-1-011 | Solicitação pending aprovada pelo designado cria uma única versão e decisão auditadas. | Transições e regras; API local | SPEC-1-011-CA-01; cenário de UI/API, releitura e evidência | Comparativo base/proposta/vigente, versão e registro de decisão. | P9 confirmada; SPEC-1-010, membros e aprovadores. | Parar se a versão-base ficou desatualizada ou executor não estiver designado. | Planejada |
| F1-27 | Resolver aprovação concorrente e transições da solicitação | Thorus | SPEC-1-011 | Segunda decisão sobre base antiga retorna 409 e não cria versão nem troca vigente. | Transições e regras; API local | SPEC-1-011-CA-02; cenário de UI/API, releitura e evidência | Duas requests concorrentes, uma v2, outra 409, timeline com uma decisão. | F1-11, F1-26 e P9. | Parar se tiver versão ou decisão órfã após falha. | Planejada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
