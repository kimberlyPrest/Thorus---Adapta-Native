# SPEC-1-014 — Aba Atividade: linha do tempo e detalhes da mudança

**Fase:** 1  
**Status:** planejada  
**Dono:** Engenharia de produto (execução) e Thórus (validação do fluxo)  
**Origem no escopo:** seção 4.1, 4.4 e 4.9; rota `/projetos/{id}/atividade`  
**Degrau da solução:** construção mínima da tela e da regra local necessárias para esta operação funcionar manualmente.

## Contexto e decisões fechadas

- **Estado atual:** não existe aplicativo executável neste repositório; a SPEC antiga correspondente está registrada no mapa de migração, commit `a359cdc`.
- **Estado desejado:** uma pessoa com o papel indicado completa a operação em `/projetos/{id}/atividade` e encontra o resultado persistido, autorizado e auditável.
- **Decisões já fechadas:** Fase 1 é manual; dados próprios ficam no banco C2; auth e grants C3/C4; UI D1–D5; nenhum conector de negócio externo nesta fase.
- **Bloqueios/dependências:** B1 para implementação de código; B3 para autenticação real. P8 para liberar usuários reais. Dependências técnicas de domínio: SPEC-1-001, SPEC-1-002, SPEC-1-004, SPEC-1-008

## Perguntas de validação do cliente

| Perguntas | O que confirmar / como responder | Referência |
|---|---|---|
| P5, P8–P9: quem pode consultar atividade e quais alterações exigem trilha. | Responder marcando opções ou com uma frase curta. Se não souber, indicar quem confirma. Para Asana, anotar nomes/valores e usar somente GET; não alterar dados nem enviar token. | [Questionário P1–P10](../../03_documentos/06-Questionario-levantamento-cliente.md) |

## Resultado observável

Negar read do recurso e comprovar retirada do evento e total na próxima consulta.

## Limites e dependências

- **Inclui:** rota `/projetos/{id}/atividade`, leitura/mutação desta SPEC, persistência pertinente, grants de servidor, estados D3, activity C3 e evidência abaixo.
- **Fora de escopo:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
- **Entradas e pré-condições:** C2–C5, UI D1–D5, grants descritos abaixo; B1 para implementação de código; B3 para autenticação real. P8 para liberar usuários reais.
- **Saídas/artefatos:** página funcional, response envelope C3, migration/constraint necessários, eventos de auditoria, evidências CA.
- **Dependências e responsáveis:** dependências listadas no cabeçalho; Produto/Engenharia fecha B1; Thórus fecha perguntas P referenciadas.
- **Atores e permissões mínimas:** activity.read + read do recurso de origem. Logs administrativos sem project_id ficam restritos ao Admin; não aparecem na aba de projeto.
- **Superfícies/arquivos/configurações afetadas:** rota, componentes próprios, endpoint/handler, tabelas C2 citadas, autorização C4 e audit event.
- **Risco e plano B:** ocultação visual não protege dados; aplicar a mesma decisão no servidor. Pendência de vocabulário usa somente fixture identificada até resposta P.
- **Rollback ou reversão:** tela pode suspender ação sem apagar histórico; migration aditiva/reversível; cancelamento preserva registro e emite auditoria.

## Página e comportamento detalhado

URL/rota: `/projetos/{id}/atividade`. Base visual única: [Contrato de interface](01-contrato-interface.md).

| Área | Conteúdo e interação |
|---|---|
| Linha do tempo | Data local, ator, verbo legível, registro e origem Manual; grupos por dia, ordem mais recente |
| Filtros | Tipo de registro, ação, autor e intervalo de data; valores possíveis só do universo autorizado |
| Detalhe | Drawer campos alterados e resumo seguro; link Ver registro se origem ainda legível |
| Histórico inacessível | Excluir eventos de recurso sem read; não vazar nomes via badge/contagem/filtro |
| Recuperação | Erro de leitura não resulta em timeline vazia; Recarregar mantém filtros |

## Dados de entrada e saída

| Campo | Regra |
|---|---|
| event | C2 append-only; project_id é escopo do evento |
| labels | resource/action traduzidos: “criou tarefa”, “aprovou alteração”, “arquivou referência” |
| details | changed_field_names e safe_summary; valor técnico integral só em versão autorizada |
| filters | date calendar inclusiva, consulta converte limites locais para UTC corretamente |

## Dados e integrações

UI chama somente API local C3; tabelas respeitam FKs/índices C2. Nenhuma API Asana/Drive é chamada nesta rota.

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Erro |
|---|---|---|---|---|---|
| Tela `/projetos/{id}/atividade` ↔ API local ↔ banco | Banco da aplicação | Campos listados em Dados de entrada; versão/C2 | activity.read + read do recurso de origem. Logs administrativos sem project_id ficam restritos ao Admin; não aparecem na aba de projeto. | C3: paginação, Idempotency-Key em escrita, expectedVersion em edição; sem retry automático de PATCH | Envelope C3, manter rascunho, correlationId |
| API local ↔ activity_events | Banco na mesma transação | ator, recurso, ação, alvo, instante UTC e resumo seguro | Herda grant da ação | Transação atômica; chave impede repetição de evento | Se log falhar, cancelar mutação e devolver erro C3 |

### Regras de negócio e regras de dados

Se ação não consegue criar evento, mutação faz rollback e UI preserva rascunho.
Evento de entidade arquivada continua legível; link leva a visão somente leitura.
Autor desativado mantém nome identificável segundo regra de retenção P8; sem e-mail pessoal desnecessário.
Não exibir conteúdo integral de mensagem/definição no log comum; histórico técnico é SPEC-1-010.

## API local desta entrega

- GET /api/projects/{id}/activity?resource=&action=&actor=&from=&to=&page= → occurred_at desc/id e total filtrados.
- GET /api/projects/{id}/activity/{eventId} → detalhe seguro filtrado por recursos; sem PATCH/DELETE de evento.
- Mutadores de cada SPEC chamam registro de atividade transacional C3; não criar auditoria apenas no frontend.

## Fluxo principal e caminhos de erro

1. Criar um registro manual já entregue e conferir evento uma única vez.
2. Filtrar tipo/ator/data e abrir detalhe seguro.
3. Negar read do recurso e comprovar retirada do evento e total na próxima consulta.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Criar um registro manual já entregue e conferir evento uma única vez. | Timeline traduz ação/ator/data e filtros retornam os eventos autorizados. | Erro de gravação mantém formulário segundo D3. |
| Permissão/limite | activity.read + read do recurso de origem. Logs administrativos sem project_id ficam restritos ao Admin; não aparecem na aba de projeto. | Commit de mutação e evento é atômico; reenvio não duplica história. | Responder 401/403/404 conforme C3, sem vazamento. |
| Falha/concorrência | Papel sem read da origem não vê evento/contador; falha de timeline não apaga nem permite editar log. | Dados permanecem íntegros, sem sucesso falso. | 409/422/503 recuperável; campos mantidos e nenhuma alteração parcial. |

## Critérios de aceite

- [ ] **SPEC-1-014-CA-01:** Timeline traduz ação/ator/data e filtros retornam os eventos autorizados.
- [ ] **SPEC-1-014-CA-02:** Commit de mutação e evento é atômico; reenvio não duplica história.
- [ ] **SPEC-1-014-CA-03:** Papel sem read da origem não vê evento/contador; falha de timeline não apaga nem permite editar log.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | SPEC-1-014-CA-01 e SPEC-1-014-CA-03 com fixture C5 | Antes da implementação, executar a ação de UI/API descrita nesta SPEC para os critérios principal, negado e conflito | Falha por ausência da regra/comportamento, sem falha de fixture; registrar status/estado observado | Saída do cenário e estado inicial fictício |
| GREEN | Todos os CA desta SPEC | Implementar a rota ponta a ponta; repetir UI/API, reler do banco e tentar request sem grant/com versão obsoleta | Cada CA passa; mutação e evento são atômicos; erro mantém rascunho | Captura por estado, resposta C3 redigida, releitura e evento |
| REFACTOR | Todos os CA desta SPEC | Reexecutar fluxo, teclado e viewports 375/768/1440; aplicar o runner escolhido em B1 quando houver código | Sem regressão; critérios continuam binários; autorização server-side e dados íntegros | Relatório focal e evidências por CA |

**Dados/fixtures:** [Contrato de dados, C5]; adicionar somente as entidades exigidas pela tabela Dados de entrada desta SPEC.  
**Caminhos de erro obrigatórios:** Negar read do recurso e comprovar retirada do evento e total na próxima consulta. Erro de permissão, referência inválida, conflito de versão e falha de banco conforme C3.  
**Evidência exigida:** um artefato por critério; captura de página com dado fictício, API response sem segredo, verificação de persistência/autorização.

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** [contrato visual](01-contrato-interface.md), [dados/API](02-contrato-dados-api.md), seção de limite/permissão desta SPEC e SPEC-1-001, SPEC-1-002, SPEC-1-004, SPEC-1-008.
2. **Alterar somente:** rota `/projetos/{id}/atividade`, componente, handler/endpoint e persistência deste fluxo.
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

- **Como demonstrar:** Negar read do recurso e comprovar retirada do evento e total na próxima consulta.
- **Como operar depois:** Thórus opera pela rota indicada usando os grants C4.
- **Como monitorar:** correlationId, falha de gravação, 401/403, conflito e evento da rota; nenhum log guarda segredo.
- **Pendência conhecida:** B1 para implementação de código; B3 para autenticação real. P8 para liberar usuários reais.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Subseção exata | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| F1-08 | Implementar atividade com auditoria transacional consultável | Thorus | SPEC-1-014 | Mutação e evento confirmam juntos; falha de auditoria desfaz a mutação. | API local; Fluxo principal e erros | SPEC-1-014-CA-01; cenário de UI/API, releitura e evidência | Timeline por ator/ação/data e simulação de rollback sem evento órfão. | C3 e primeira rota mutável concluída. | Parar se logger registrar segredo/texto técnico integral ou aceitar mutação sem evento. | Planejada |
| F1-30 | Filtrar atividade e retirar eventos sem permissão de origem | Thorus | SPEC-1-014 | Filtros/totais exibem somente eventos de recursos com read; rollback nunca deixa evento sem mutação. | API local; Estados e erros | SPEC-1-014-CA-02; cenário de UI/API, releitura e evidência | Roteiro CS/admin, filtros por data/ator, simulação de erro transacional. | F1-08, F1-22 e C4. | Parar se log editável/excluível ou campo sensível estiver exposto. | Planejada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
