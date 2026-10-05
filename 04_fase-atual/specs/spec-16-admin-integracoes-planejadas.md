# SPEC-1-016 — Administração de mapeamentos preparados para Fase 2

**Fase:** 1  
**Status:** planejada  
**Dono:** Engenharia de produto (execução) e Thórus (validação do fluxo)  
**Origem no escopo:** seção 4.2, 4.3 e 4.9; rota `/configuracoes/integracoes`  
**Degrau da solução:** construção mínima da tela e da regra local necessárias para esta operação funcionar manualmente.

## Contexto e decisões fechadas

- **Estado atual:** não existe aplicativo executável neste repositório; a SPEC antiga correspondente está registrada no mapa de migração, commit `a359cdc`.
- **Estado desejado:** uma pessoa com o papel indicado completa a operação em `/configuracoes/integracoes` e encontra o resultado persistido, autorizado e auditável.
- **Decisões já fechadas:** Fase 1 é manual; dados próprios ficam no banco C2; auth e grants C3/C4; UI D1–D5; nenhum conector de negócio externo nesta fase.
- **Bloqueios/dependências:** P1/P2/P3 para revisar mapeamento real; configurar rascunho manual é demonstrável com GIDs fictícios. Dependências técnicas de domínio: SPEC-1-001, SPEC-1-002, SPEC-1-004, SPEC-1-007

## Perguntas de validação do cliente

| Perguntas | O que confirmar / como responder | Referência |
|---|---|---|
| P1–P4: sistemas, dados e acessos a preparar; integração só na Fase 2. | Responder marcando opções ou com uma frase curta. Se não souber, indicar quem confirma. Para Asana, anotar nomes/valores e usar somente GET; não alterar dados nem enviar token. | [Questionário P1–P10](../../03_documentos/06-Questionario-levantamento-cliente.md) |

## Resultado observável

Recarrega tela e comprova enabled=false e ausência de chamada ao serviço externo.

## Limites e dependências

- **Inclui:** rota `/configuracoes/integracoes`, leitura/mutação desta SPEC, persistência pertinente, grants de servidor, estados D3, activity C3 e evidência abaixo.
- **Fora de escopo:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
- **Entradas e pré-condições:** C2–C5, UI D1–D5, grants descritos abaixo; P1/P2/P3 para revisar mapeamento real; configurar rascunho manual é demonstrável com GIDs fictícios.
- **Saídas/artefatos:** página funcional, response envelope C3, migration/constraint necessários, eventos de auditoria, evidências CA.
- **Dependências e responsáveis:** dependências listadas no cabeçalho; Produto/Engenharia fecha B1; Thórus fecha perguntas P referenciadas.
- **Atores e permissões mínimas:** Admin com admin_mappings.read/update. Endpoints armazenam configuração local; nenhum worker/conector de rede registrado.
- **Superfícies/arquivos/configurações afetadas:** rota, componentes próprios, endpoint/handler, tabelas C2 citadas, autorização C4 e audit event.
- **Risco e plano B:** ocultação visual não protege dados; aplicar a mesma decisão no servidor. Pendência de vocabulário usa somente fixture identificada até resposta P.
- **Rollback ou reversão:** tela pode suspender ação sem apagar histórico; migration aditiva/reversível; cancelamento preserva registro e emite auditoria.

## Página e comportamento detalhado

URL/rota: `/configuracoes/integracoes`. Base visual única: [Contrato de interface](01-contrato-interface.md).

| Área | Conteúdo e interação |
|---|---|
| Header | Integrações planejadas; banner “Conexão prevista para Fase 2” |
| Seleção | Asana / Drive; dados planejados e perguntas relevantes visíveis |
| Mapeamentos | Sistema/Recurso/Campo externo/Campo interno/Transformação/Revisão; editar draft e marcar revisão humana |
| Vínculos | Selecionar projeto, informar workspace/GID/tipo/permalink opcionais, origem manual visível |
| Limite de UI | Sem botão Conectar/Sincronizar/Testar/Upload, sem input token/secret; Fase 1 mantém enabled=false |

## Dados de entrada e saída

| Campo | Regra |
|---|---|
| system/resource/externalField | system asana/drive; resource allowlist project/task/document/folder; chaves externas texto ≤120 |
| internalField | Allowlist de campos simples C2; não permitir current_version_id, usuário, grant ou estado decidido por importação |
| transform/enumMap | identity/text/date/enum_mapping; enum map requer pares explícitos, sem código arbitrário |
| reviewState/revision/enabled | draft/reviewed; incremento revision; enabled sempre false |
| project link | GID/workspace/permalink opcionais; C2 unicidade; não solicitar segredo |

## Dados e integrações

UI chama somente API local C3; tabelas respeitam FKs/índices C2. Nenhuma API Asana/Drive é chamada nesta rota.

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Erro |
|---|---|---|---|---|---|
| Tela `/configuracoes/integracoes` ↔ API local ↔ banco | Banco da aplicação | Campos listados em Dados de entrada; versão/C2 | Admin com admin_mappings.read/update. Endpoints armazenam configuração local; nenhum worker/conector de rede registrado. | C3: paginação, Idempotency-Key em escrita, expectedVersion em edição; sem retry automático de PATCH | Envelope C3, manter rascunho, correlationId |
| API local ↔ activity_events | Banco na mesma transação | ator, recurso, ação, alvo, instante UTC e resumo seguro | Herda grant da ação | Transação atômica; chave impede repetição de evento | Se log falhar, cancelar mutação e devolver erro C3 |

### Regras de negócio e regras de dados

Campos externos não sobrescrevem manual; revisar mapeamento não executa transformação/importação.
Configuração com GID duplicado no mesmo sistema/workspace/tipo recusa; null permitido.
P1/P2/P3 preparam a Fase 2; ausência de resposta não impede CRUD do projeto manual.
Contrato de retorno futuro: success/partial/failed, correlationId, counts, errors, sourceUpdatedAt/syncedAt; não emitir fake syncedAt na Fase 1.

## API local desta entrega

- GET/POST /api/admin/integration-mappings; PATCH /{id} {campos,expectedVersion}; payload enabled=true retorna 422.
- GET/POST /api/admin/project-external-links; PATCH /{id} {workspaceId,gid,permalink,expectedVersion}.
- Interfaces futuras reader/document store descritas apenas no contrato; nesta fase adapter rejeita operação como INTEGRATION_NOT_ENABLED sem abrir socket.

## Fluxo principal e caminhos de erro

1. Admin escolhe sistema e cadastra mapeamento externo→interno como draft.
2. Revisa e salva versão; cadastra vínculo de um projeto manual.
3. Recarrega tela e comprova enabled=false e ausência de chamada ao serviço externo.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Admin escolhe sistema e cadastra mapeamento externo→interno como draft. | Mapeamento/vínculo persistem com revisão/IDs opcionais sem bloquear uso manual. | Erro de gravação mantém formulário segundo D3. |
| Permissão/limite | Admin com admin_mappings.read/update. Endpoints armazenam configuração local; nenhum worker/conector de rede registrado. | enabled=true, campo interno protegido, código transform e GID duplicado são rejeitados. | Responder 401/403/404 conforme C3, sem vazamento. |
| Falha/concorrência | Todas ações só chamam API local; nenhuma integração é conectada ou solicita segredo. | Dados permanecem íntegros, sem sucesso falso. | 409/422/503 recuperável; campos mantidos e nenhuma alteração parcial. |

## Critérios de aceite

- [ ] **SPEC-1-016-CA-01:** Mapeamento/vínculo persistem com revisão/IDs opcionais sem bloquear uso manual.
- [ ] **SPEC-1-016-CA-02:** enabled=true, campo interno protegido, código transform e GID duplicado são rejeitados.
- [ ] **SPEC-1-016-CA-03:** Todas ações só chamam API local; nenhuma integração é conectada ou solicita segredo.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | SPEC-1-016-CA-01 e SPEC-1-016-CA-03 com fixture C5 | Antes da implementação, executar a ação de UI/API descrita nesta SPEC para os critérios principal, negado e conflito | Falha por ausência da regra/comportamento, sem falha de fixture; registrar status/estado observado | Saída do cenário e estado inicial fictício |
| GREEN | Todos os CA desta SPEC | Implementar a rota ponta a ponta; repetir UI/API, reler do banco e tentar request sem grant/com versão obsoleta | Cada CA passa; mutação e evento são atômicos; erro mantém rascunho | Captura por estado, resposta C3 redigida, releitura e evento |
| REFACTOR | Todos os CA desta SPEC | Reexecutar fluxo, teclado e viewports 375/768/1440; aplicar o runner escolhido em B1 quando houver código | Sem regressão; critérios continuam binários; autorização server-side e dados íntegros | Relatório focal e evidências por CA |

**Dados/fixtures:** [Contrato de dados, C5]; adicionar somente as entidades exigidas pela tabela Dados de entrada desta SPEC.  
**Caminhos de erro obrigatórios:** Recarrega tela e comprova enabled=false e ausência de chamada ao serviço externo. Erro de permissão, referência inválida, conflito de versão e falha de banco conforme C3.  
**Evidência exigida:** um artefato por critério; captura de página com dado fictício, API response sem segredo, verificação de persistência/autorização.

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** [contrato visual](01-contrato-interface.md), [dados/API](02-contrato-dados-api.md), seção de limite/permissão desta SPEC e SPEC-1-001, SPEC-1-002, SPEC-1-004, SPEC-1-007.
2. **Alterar somente:** rota `/configuracoes/integracoes`, componente, handler/endpoint e persistência deste fluxo.
3. **Não alterar:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
4. **Executar nesta ordem:** migration/contrato; leitura autorizada; ação principal e commit; estado de erro; teclado/mobile; captura de evidência.
5. **Parar e pedir validação quando:** P1/P2/P3 para revisar mapeamento real; configurar rascunho manual é demonstrável com GIDs fictícios.
6. **Estado válido ao parar:** rota principal não grava parcial; sessão/permissões protegem resposta; dados existentes e histórico continuam íntegros.

## Checklist de execução

- [ ] Grant e lista de campos confirmados; fixture mínima tem chave/teste repetível.
- [ ] Caminho principal liga tela, servidor, banco e evento de auditoria na mesma entrega.
- [ ] Vazio, erro, inválido, negado, concorrência e sucesso aplicam D3 na rota.
- [ ] Teclado/foco/viewport e evidências por CA verificados.
- [ ] Thórus confere o fluxo com dados fictícios; pendência de produto registrada.

## Handoff e operação

- **Como demonstrar:** Recarrega tela e comprova enabled=false e ausência de chamada ao serviço externo.
- **Como operar depois:** Thórus opera pela rota indicada usando os grants C4.
- **Como monitorar:** correlationId, falha de gravação, 401/403, conflito e evento da rota; nenhum log guarda segredo.
- **Pendência conhecida:** P1/P2/P3 para revisar mapeamento real; configurar rascunho manual é demonstrável com GIDs fictícios.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Subseção exata | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| F1-09 | Preparar mapeamentos e GIDs externos sem integração ativa | Thorus | SPEC-1-016 | Admin salva mapeamento e vínculo opcional; runtime permanece enabled=false e sem egress Asana/Drive. | Página e comportamento detalhado; Limites | SPEC-1-016-CA-01; cenário de UI/API, releitura e evidência | Captura de config persistida, GID duplicado recusado e inspeção de chamadas externas. | P1/P2/P3, C2 e papel admin_mappings. | Parar se houver token, conexão, sync, upload ou chamada de rede externa. | Planejada |
| F1-33 | Implementar mapeamento draft/review e IDs externos opcionais | Thorus | SPEC-1-016 | Configuração Asana/Drive é salva e lida com revision; serviço segue enabled=false. | Página e comportamento detalhado | SPEC-1-016-CA-02; cenário de UI/API, releitura e evidência | Capturas, API local, duplicidade C2 e busca por socket/network. | P1/P2/P3, F1-03/F1-15 e permission Admin. | Parar se input/segredo ou estado “conectado” sugerir integração. | Planejada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
