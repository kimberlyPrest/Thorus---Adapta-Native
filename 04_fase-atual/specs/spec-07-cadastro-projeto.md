# SPEC-1-007 — Novo e editar projeto com cliente e salvamento

**Fase:** 1  
**Status:** planejada  
**Dono:** Engenharia de produto (execução) e Thórus (validação do fluxo)  
**Origem no escopo:** seção 4.1 e 4.9; rota `/projetos/novo e /projetos/{id}/editar`  
**Degrau da solução:** construção mínima da tela e da regra local necessárias para esta operação funcionar manualmente.

## Contexto e decisões fechadas

- **Estado atual:** não existe aplicativo executável neste repositório; a SPEC antiga correspondente está registrada no mapa de migração, commit `a359cdc`.
- **Estado desejado:** uma pessoa com o papel indicado completa a operação em `/projetos/novo e /projetos/{id}/editar` e encontra o resultado persistido, autorizado e auditável.
- **Decisões já fechadas:** Fase 1 é manual; dados próprios ficam no banco C2; auth e grants C3/C4; UI D1–D5; nenhum conector de negócio externo nesta fase.
- **Bloqueios/dependências:** B1 antes da migração/primeira criação; P1/P2 validam dados reais. MVP fictício usa mínimos C2. Dependências técnicas de domínio: SPEC-1-001, SPEC-1-002, SPEC-1-004, SPEC-1-005

## Perguntas de validação do cliente

| Perguntas | O que confirmar / como responder | Referência |
|---|---|---|
| P1–P2: amostra de projetos/campos e qual fonte prevalece; deixar integração desligada na Fase 1. | Responder marcando opções ou com uma frase curta. Se não souber, indicar quem confirma. Para Asana, anotar nomes/valores e usar somente GET; não alterar dados nem enviar token. | [Questionário P1–P10](../../03_documentos/06-Questionario-levantamento-cliente.md) |

## Resultado observável

Editar muda escopo/data; concorrência compara versão e mantém rascunho.

## Limites e dependências

- **Inclui:** rota `/projetos/novo e /projetos/{id}/editar`, leitura/mutação desta SPEC, persistência pertinente, grants de servidor, estados D3, activity C3 e evidência abaixo.
- **Fora de escopo:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
- **Entradas e pré-condições:** C2–C5, UI D1–D5, grants descritos abaixo; B1 antes da migração/primeira criação; P1/P2 validam dados reais. MVP fictício usa mínimos C2.
- **Saídas/artefatos:** página funcional, response envelope C3, migration/constraint necessários, eventos de auditoria, evidências CA.
- **Dependências e responsáveis:** dependências listadas no cabeçalho; Produto/Engenharia fecha B1; Thórus fecha perguntas P referenciadas.
- **Atores e permissões mínimas:** projects.create para novo; projects.update + escopo para edição. Cliente novo por ação de cadastro de projeto autorizada; listar cliente respeita universo C4.
- **Superfícies/arquivos/configurações afetadas:** rota, componentes próprios, endpoint/handler, tabelas C2 citadas, autorização C4 e audit event.
- **Risco e plano B:** ocultação visual não protege dados; aplicar a mesma decisão no servidor. Pendência de vocabulário usa somente fixture identificada até resposta P.
- **Rollback ou reversão:** tela pode suspender ação sem apagar histórico; migration aditiva/reversível; cancelamento preserva registro e emite auditoria.

## Página e comportamento detalhado

URL/rota: `/projetos/novo e /projetos/{id}/editar`. Base visual única: [Contrato de interface](01-contrato-interface.md).

| Área | Conteúdo e interação |
|---|---|
| Formulário | Página dedicada, não modal; grupos Identificação, Responsáveis, Planejamento, Observações; grid duas colunas desktop/uma mobile |
| Identificação | Nome, Código opcional, Cliente pesquisável, botão “Novo cliente” abre drawer só Nome, Município/UF, Escopo contratado |
| Responsáveis | CS e responsável técnico opcionais; selects de usuários ativos; atribuí-los cria memberships no mesmo commit |
| Planejamento | Fase/situação dos catálogos, início e previsão de entrega opcionais |
| Footer | “Salvar projeto” / “Cancelar”; dirty guard; sucesso abre /projetos/{id} |
| Edição | Campos preenchidos, fonte Manual, updated_at/autor; 409 abre comparação autorizada D3 |

## Dados de entrada e saída

| Campo | Regra |
|---|---|
| name/client_id | Obrigatórios; limites C2; client existente ou criado no drawer antes de salvar projeto |
| code | Opcional único; trim/uppercase; duplicidade identifica só registro acessível |
| city/state_code/contracted_scope | Opcionais e limites C2; UF válida; sem dados pessoais obrigatórios |
| cs_user_id/technical_user_id | Opcionais; ativos; após seleção entram como membros; papel de login não muda |
| phase/status/dates/notes | Opcionais; ativos para nova seleção; manter inativo anterior em edição se não mudar |
| expectedVersion | Edição exige versão atual |

## Dados e integrações

UI chama somente API local C3; tabelas respeitam FKs/índices C2. Nenhuma API Asana/Drive é chamada nesta rota.

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Erro |
|---|---|---|---|---|---|
| Tela `/projetos/novo e /projetos/{id}/editar` ↔ API local ↔ banco | Banco da aplicação | Campos listados em Dados de entrada; versão/C2 | projects.create para novo; projects.update + escopo para edição. Cliente novo por ação de cadastro de projeto autorizada; listar cliente respeita universo C4. | C3: paginação, Idempotency-Key em escrita, expectedVersion em edição; sem retry automático de PATCH | Envelope C3, manter rascunho, correlationId |
| API local ↔ activity_events | Banco na mesma transação | ator, recurso, ação, alvo, instante UTC e resumo seguro | Herda grant da ação | Transação atômica; chave impede repetição de evento | Se log falhar, cancelar mutação e devolver erro C3 |

### Regras de negócio e regras de dados

Campo obrigatório mínimo é nome + cliente. Restante aceita “Não informado” até P1/P2; mudanças de obrigatoriedade precisam emenda de produto, não toggle irrestrito.
Criador com escopo assigned sempre fica como membro para não perder acesso após criar.
Cliente novo pode ter nome igual a existente: UI sugere já existente por normalização mas não afirma identidade só pelo nome. Sem deduplicação destrutiva.
Projeto e vínculos não podem ficar meio gravados; sucesso exige commit completo. Query de edição não inclui projetos fora do acesso.

## API local desta entrega

- GET /api/projects/{id}/edit e GET /api/projects/form-options → campos/valores autorizados; lookup de usuários retorna somente id/name/role.
- POST /api/clients {name} → cliente local; retry idempotente evita duplicata do mesmo clique.
- POST /api/projects {name,code,clientId,city,stateCode,contractedScope,csUserId,technicalUserId,phaseOptionId,statusOptionId,startDate,targetDate,notes} → projeto + creator membership + membros escolhidos + auditoria atômicos.
- PATCH /api/projects/{id} {campos permitidos,expectedVersion} → projeto novo snapshot; não excluir membro antigo automaticamente ao trocar responsável (gestão na SPEC-1-008).

## Fluxo principal e caminhos de erro

1. CS abre Novo, seleciona/cria cliente e preenche nome + demais campos desejados.
2. Salvar persiste e abre Visão geral; logout/login confirma dados.
3. Editar muda escopo/data; concorrência compara versão e mantém rascunho.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | CS abre Novo, seleciona/cria cliente e preenche nome + demais campos desejados. | Nome/cliente suficientes para criar projeto e abrir detalhe persistido com creator membership. | Erro de gravação mantém formulário segundo D3. |
| Permissão/limite | projects.create para novo; projects.update + escopo para edição. Cliente novo por ação de cadastro de projeto autorizada; listar cliente respeita universo C4. | Editar respeita autorização, versionamento, datas e dados opcionais não informados. | Responder 401/403/404 conforme C3, sem vazamento. |
| Falha/concorrência | Código duplicado, usuário inativo, data invertida, duplo clique e banco indisponível não produzem projeto parcial. | Dados permanecem íntegros, sem sucesso falso. | 409/422/503 recuperável; campos mantidos e nenhuma alteração parcial. |

## Critérios de aceite

- [ ] **SPEC-1-007-CA-01:** Nome/cliente suficientes para criar projeto e abrir detalhe persistido com creator membership.
- [ ] **SPEC-1-007-CA-02:** Editar respeita autorização, versionamento, datas e dados opcionais não informados.
- [ ] **SPEC-1-007-CA-03:** Código duplicado, usuário inativo, data invertida, duplo clique e banco indisponível não produzem projeto parcial.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | SPEC-1-007-CA-01 e SPEC-1-007-CA-03 com fixture C5 | Antes da implementação, executar a ação de UI/API descrita nesta SPEC para os critérios principal, negado e conflito | Falha por ausência da regra/comportamento, sem falha de fixture; registrar status/estado observado | Saída do cenário e estado inicial fictício |
| GREEN | Todos os CA desta SPEC | Implementar a rota ponta a ponta; repetir UI/API, reler do banco e tentar request sem grant/com versão obsoleta | Cada CA passa; mutação e evento são atômicos; erro mantém rascunho | Captura por estado, resposta C3 redigida, releitura e evento |
| REFACTOR | Todos os CA desta SPEC | Reexecutar fluxo, teclado e viewports 375/768/1440; aplicar o runner escolhido em B1 quando houver código | Sem regressão; critérios continuam binários; autorização server-side e dados íntegros | Relatório focal e evidências por CA |

**Dados/fixtures:** [Contrato de dados, C5]; adicionar somente as entidades exigidas pela tabela Dados de entrada desta SPEC.  
**Caminhos de erro obrigatórios:** Editar muda escopo/data; concorrência compara versão e mantém rascunho. Erro de permissão, referência inválida, conflito de versão e falha de banco conforme C3.  
**Evidência exigida:** um artefato por critério; captura de página com dado fictício, API response sem segredo, verificação de persistência/autorização.

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** [contrato visual](01-contrato-interface.md), [dados/API](02-contrato-dados-api.md), seção de limite/permissão desta SPEC e SPEC-1-001, SPEC-1-002, SPEC-1-004, SPEC-1-005.
2. **Alterar somente:** rota `/projetos/novo e /projetos/{id}/editar`, componente, handler/endpoint e persistência deste fluxo.
3. **Não alterar:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
4. **Executar nesta ordem:** migration/contrato; leitura autorizada; ação principal e commit; estado de erro; teclado/mobile; captura de evidência.
5. **Parar e pedir validação quando:** B1 antes da migração/primeira criação; P1/P2 validam dados reais. MVP fictício usa mínimos C2.
6. **Estado válido ao parar:** rota principal não grava parcial; sessão/permissões protegem resposta; dados existentes e histórico continuam íntegros.

## Checklist de execução

- [ ] Grant e lista de campos confirmados; fixture mínima tem chave/teste repetível.
- [ ] Caminho principal liga tela, servidor, banco e evento de auditoria na mesma entrega.
- [ ] Vazio, erro, inválido, negado, concorrência e sucesso aplicam D3 na rota.
- [ ] Teclado/foco/viewport e evidências por CA verificados.
- [ ] Thórus confere o fluxo com dados fictícios; pendência de produto registrada.

## Handoff e operação

- **Como demonstrar:** Editar muda escopo/data; concorrência compara versão e mantém rascunho.
- **Como operar depois:** Thórus opera pela rota indicada usando os grants C4.
- **Como monitorar:** correlationId, falha de gravação, 401/403, conflito e evento da rota; nenhum log guarda segredo.
- **Pendência conhecida:** B1 antes da migração/primeira criação; P1/P2 validam dados reais. MVP fictício usa mínimos C2.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Subseção exata | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| F1-01 | Confirmar e aprovar campos mínimos do cadastro manual | Thorus | SPEC-1-007 | Produto e Thórus aprovam o dicionário de campos C2 aplicável ao projeto piloto. | Decisões de campos e obrigatoriedade | SPEC-1-007-CA-01; cenário de UI/API, releitura e evidência | Dicionário com campo, tipo, obrigatoriedade, permissão e exemplo fictício aprovado. | P1/P2 respondidas para dados do projeto; campo pessoal sem finalidade não entra. | Parar ao encontrar campo obrigatório sem finalidade, origem ou acesso definido. | Planejada |
| F1-03 | Fechar o modelo relacional do projeto e clientes no contrato C2 | Thorus | SPEC-1-007 | C2 para clients/projects e relacionamentos fica aprovado para migration; B1 runtime/db/runner documentados. | Modelo de dados da rota | SPEC-1-007-CA-02; cenário de UI/API, releitura e evidência | Modelo relacional e constraint list de clients/projects versionados. | Acesso ao repositório executável e decisão técnica B1. | Parar sem escolher engine/provider por suposição. | Planejada |
| F1-15 | Persistir cadastro de clientes e projetos a partir dos formulários | Thorus | SPEC-1-007 | Cadastro manual completo grava client, projeto, memberships e evento na transação C3. | API local e modelo de dados; Fluxo | SPEC-1-007-CA-03; cenário de UI/API, releitura e evidência | Migration aplicada em banco vazio e releitura após sair/entrar. | F1-01/F1-03, B1 e formulário acessível. | Parar em migration que perca dados, ausência de backup local ou gravação parcial. | Planejada |
| F1-23 | Implementar edição concorrente e falha de gravação no cadastro | Thorus | SPEC-1-007 | Conflito mostra versão atual × rascunho e erro de banco mantém campos sem persistência parcial. | Fluxo principal e erros; C3 | SPEC-1-007-CA-03; cenário de UI/API, releitura e evidência | Prova 409, versão, formulários e duas linhas transacionais. | F1-15 e C3 versioning/idempotency. | Parar se retry automático sobrescrever versão concorrente. | Planejada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
