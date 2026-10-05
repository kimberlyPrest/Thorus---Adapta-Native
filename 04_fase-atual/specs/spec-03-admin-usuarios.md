# SPEC-1-003 — Administração de usuários pela interface

**Fase:** 1  
**Status:** planejada  
**Dono:** Engenharia de produto (execução) e Thórus (validação do fluxo)  
**Origem no escopo:** seção 3 e 4.9; rota `/configuracoes/usuarios`  
**Degrau da solução:** construção mínima da tela e da regra local necessárias para esta operação funcionar manualmente.

## Contexto e decisões fechadas

- **Estado atual:** não existe aplicativo executável neste repositório; a SPEC antiga correspondente está registrada no mapa de migração, commit `a359cdc`.
- **Estado desejado:** uma pessoa com o papel indicado completa a operação em `/configuracoes/usuarios` e encontra o resultado persistido, autorizado e auditável.
- **Decisões já fechadas:** Fase 1 é manual; dados próprios ficam no banco C2; auth e grants C3/C4; UI D1–D5; nenhum conector de negócio externo nesta fase.
- **Bloqueios/dependências:** B1/B3 e P8 antes de usuários reais. Para demonstração, contas fixture e cadastro local pending são suficientes. Dependências técnicas de domínio: SPEC-1-001, SPEC-1-002, SPEC-1-004

## Perguntas de validação do cliente

| Perguntas | O que confirmar / como responder | Referência |
|---|---|---|
| P8: quem acessa projetos/documentos e como vincular a pessoa ao projeto. | Responder marcando opções ou com uma frase curta. Se não souber, indicar quem confirma. Para Asana, anotar nomes/valores e usar somente GET; não alterar dados nem enviar token. | [Questionário P1–P10](../../03_documentos/06-Questionario-levantamento-cliente.md) |

## Resultado observável

Admin edita vínculo, desativa usuário de teste e verifica que sua próxima ação é recusada.

## Limites e dependências

- **Inclui:** rota `/configuracoes/usuarios`, leitura/mutação desta SPEC, persistência pertinente, grants de servidor, estados D3, activity C3 e evidência abaixo.
- **Fora de escopo:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
- **Entradas e pré-condições:** C2–C5, UI D1–D5, grants descritos abaixo; B1/B3 e P8 antes de usuários reais. Para demonstração, contas fixture e cadastro local pending são suficientes.
- **Saídas/artefatos:** página funcional, response envelope C3, migration/constraint necessários, eventos de auditoria, evidências CA.
- **Dependências e responsáveis:** dependências listadas no cabeçalho; Produto/Engenharia fecha B1; Thórus fecha perguntas P referenciadas.
- **Atores e permissões mínimas:** Admin com admin_users.read/create/update/deactivate. Cadastro usa o perfil autorizado e vínculos de projetos; demais papéis recebem 403 no endpoint.
- **Superfícies/arquivos/configurações afetadas:** rota, componentes próprios, endpoint/handler, tabelas C2 citadas, autorização C4 e audit event.
- **Risco e plano B:** ocultação visual não protege dados; aplicar a mesma decisão no servidor. Pendência de vocabulário usa somente fixture identificada até resposta P.
- **Rollback ou reversão:** tela pode suspender ação sem apagar histórico; migration aditiva/reversível; cancelamento preserva registro e emite auditoria.

## Página e comportamento detalhado

URL/rota: `/configuracoes/usuarios`. Base visual única: [Contrato de interface](01-contrato-interface.md).

| Área | Conteúdo e interação |
|---|---|
| Header | “Usuários”, contador, CTA “Novo usuário” |
| Filtros/tabela | Busca nome/e-mail, perfil, ativo/inativo/pendente; colunas Nome, E-mail, Perfil, Acesso, Projetos atribuídos e Ações |
| Novo usuário | Drawer: Nome, E-mail corporativo, Perfil, projetos atribuídos; “Salvar usuário”. Estado inicial Pendente de identidade |
| Editar | Mesmo drawer + vínculo de identidade somente pelo processo seguro B3; após vincular, e-mail de identidade imutável na UI |
| Desativar/reativar | Confirmar consequência; badge Inativo após resposta; histórico permanece |
| Sem usuários de filtro | Limpar filtros; conta Admin inicial existe por bootstrap seguro registrado, sem seed de senha versionado |

## Dados de entrada e saída

| Campo | Obrigatório/validação |
|---|---|
| name/email | C2; e-mail trim/lowercase, formato válido; duplicidade não cria segunda conta |
| role_id | Um dos cinco perfis; obrigatório |
| memberships[] | IDs de projetos autorizados; lista opcional, escopo assigned sem vínculo não acessa projeto |
| active/access_state | Inicial active=true/pending; Admin não transforma cadastro local em identidade corporativa |
| expectedVersion | Obrigatório editar/desativar; C3 |

## Dados e integrações

UI chama somente API local C3; tabelas respeitam FKs/índices C2. Nenhuma API Asana/Drive é chamada nesta rota.

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Erro |
|---|---|---|---|---|---|
| Tela `/configuracoes/usuarios` ↔ API local ↔ banco | Banco da aplicação | Campos listados em Dados de entrada; versão/C2 | Admin com admin_users.read/create/update/deactivate. Cadastro usa o perfil autorizado e vínculos de projetos; demais papéis recebem 403 no endpoint. | C3: paginação, Idempotency-Key em escrita, expectedVersion em edição; sem retry automático de PATCH | Envelope C3, manter rascunho, correlationId |
| API local ↔ activity_events | Banco na mesma transação | ator, recurso, ação, alvo, instante UTC e resumo seguro | Herda grant da ação | Transação atômica; chave impede repetição de evento | Se log falhar, cancelar mutação e devolver erro C3 |

### Regras de negócio e regras de dados

Cadastro local não envia convite nem cria conta no IdP. Exibir próximo passo “Vincular identidade no fluxo corporativo aprovado”.
Não permitir auto-desativação nem desativação do último Admin ativo; resposta 409 com motivo.
Troca de perfil/atribuição muda capacidades já na próxima requisição, sem apagar autoria histórica.
Lista de projetos retorna somente título/ID necessários para gestão; falha de gravação mantém drawer e todas as seleções.

## API local desta entrega

- GET /api/admin/users?q=&role=&active=&accessState=&page=&pageSize= → lista local paginada e total.
- POST /api/admin/users {name,email,roleId,projectIds} → 201 usuário pending; user + memberships + evento atômicos.
- PATCH /api/admin/users/{id} {name,roleId,projectIds,expectedVersion}; e-mail só editável enquanto sem subject.
- POST /api/admin/users/{id}/deactivate ou /activate {expectedVersion}; vínculo subject segue fluxo de B3 com identidade verificada.

## Fluxo principal e caminhos de erro

1. Admin abre lista, filtra, abre Novo usuário, informa nome/e-mail/perfil e projetos.
2. Salvar cria usuário local Pendente e atualiza lista; vinculação B3 validada torna ready.
3. Admin edita vínculo, desativa usuário de teste e verifica que sua próxima ação é recusada.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Admin abre lista, filtra, abre Novo usuário, informa nome/e-mail/perfil e projetos. | Novo usuário aparece como pending com perfil e projetos persistidos; segundo submit não duplica. | Erro de gravação mantém formulário segundo D3. |
| Permissão/limite | Admin com admin_users.read/create/update/deactivate. Cadastro usa o perfil autorizado e vínculos de projetos; demais papéis recebem 403 no endpoint. | Editar/ativar/desativar funciona na UI e no banco; usuário inativo não acessa dados. | Responder 401/403/404 conforme C3, sem vazamento. |
| Falha/concorrência | E-mail duplicado, último Admin, perfil indevido, 409 e falha de banco têm mensagens específicas e preservam formulário. | Dados permanecem íntegros, sem sucesso falso. | 409/422/503 recuperável; campos mantidos e nenhuma alteração parcial. |

## Critérios de aceite

- [ ] **SPEC-1-003-CA-01:** Novo usuário aparece como pending com perfil e projetos persistidos; segundo submit não duplica.
- [ ] **SPEC-1-003-CA-02:** Editar/ativar/desativar funciona na UI e no banco; usuário inativo não acessa dados.
- [ ] **SPEC-1-003-CA-03:** E-mail duplicado, último Admin, perfil indevido, 409 e falha de banco têm mensagens específicas e preservam formulário.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | SPEC-1-003-CA-01 e SPEC-1-003-CA-03 com fixture C5 | Antes da implementação, executar a ação de UI/API descrita nesta SPEC para os critérios principal, negado e conflito | Falha por ausência da regra/comportamento, sem falha de fixture; registrar status/estado observado | Saída do cenário e estado inicial fictício |
| GREEN | Todos os CA desta SPEC | Implementar a rota ponta a ponta; repetir UI/API, reler do banco e tentar request sem grant/com versão obsoleta | Cada CA passa; mutação e evento são atômicos; erro mantém rascunho | Captura por estado, resposta C3 redigida, releitura e evento |
| REFACTOR | Todos os CA desta SPEC | Reexecutar fluxo, teclado e viewports 375/768/1440; aplicar o runner escolhido em B1 quando houver código | Sem regressão; critérios continuam binários; autorização server-side e dados íntegros | Relatório focal e evidências por CA |

**Dados/fixtures:** [Contrato de dados, C5]; adicionar somente as entidades exigidas pela tabela Dados de entrada desta SPEC.  
**Caminhos de erro obrigatórios:** Admin edita vínculo, desativa usuário de teste e verifica que sua próxima ação é recusada. Erro de permissão, referência inválida, conflito de versão e falha de banco conforme C3.  
**Evidência exigida:** um artefato por critério; captura de página com dado fictício, API response sem segredo, verificação de persistência/autorização.

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** [contrato visual](01-contrato-interface.md), [dados/API](02-contrato-dados-api.md), seção de limite/permissão desta SPEC e SPEC-1-001, SPEC-1-002, SPEC-1-004.
2. **Alterar somente:** rota `/configuracoes/usuarios`, componente, handler/endpoint e persistência deste fluxo.
3. **Não alterar:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
4. **Executar nesta ordem:** migration/contrato; leitura autorizada; ação principal e commit; estado de erro; teclado/mobile; captura de evidência.
5. **Parar e pedir validação quando:** B1/B3 e P8 antes de usuários reais. Para demonstração, contas fixture e cadastro local pending são suficientes.
6. **Estado válido ao parar:** rota principal não grava parcial; sessão/permissões protegem resposta; dados existentes e histórico continuam íntegros.

## Checklist de execução

- [ ] Grant e lista de campos confirmados; fixture mínima tem chave/teste repetível.
- [ ] Caminho principal liga tela, servidor, banco e evento de auditoria na mesma entrega.
- [ ] Vazio, erro, inválido, negado, concorrência e sucesso aplicam D3 na rota.
- [ ] Teclado/foco/viewport e evidências por CA verificados.
- [ ] Thórus confere o fluxo com dados fictícios; pendência de produto registrada.

## Handoff e operação

- **Como demonstrar:** Admin edita vínculo, desativa usuário de teste e verifica que sua próxima ação é recusada.
- **Como operar depois:** Thórus opera pela rota indicada usando os grants C4.
- **Como monitorar:** correlationId, falha de gravação, 401/403, conflito e evento da rota; nenhum log guarda segredo.
- **Pendência conhecida:** B1/B3 e P8 antes de usuários reais. Para demonstração, contas fixture e cadastro local pending são suficientes.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Subseção exata | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| F1-14 | Entregar lista e fluxo de criação e edição de usuários Admin | Thorus | SPEC-1-003 | Admin cria, encontra, edita e desativa/reativa usuário; perfil/membros ficam registrados. | Página e comportamento detalhado; API local | SPEC-1-003-CA-01; cenário de UI/API, releitura e evidência | Roteiro da tabela, drawer e nova sessão após desativação; auditoria. | P8 e B1/B3 identificados; usar fixtures para pessoa ainda sem identity. | Parar ao atingir último Admin, auto-desativação ou risco de conceder acesso indevido. | Planejada |
| F1-18 | Implementar validações e proteção do ciclo de vida do usuário | Thorus | SPEC-1-003 | E-mail duplicado/último Admin não gera gravação; edição concorrente preserva rascunho. | Modelo de dados e regras de ciclo | SPEC-1-003-CA-02; cenário de UI/API, releitura e evidência | Evidências 409/422, drawer preenchido, nome e-mail normalizados. | F1-14 e C2/C3. | Parar se cadastro local for apresentado como identidade criada no IdP. | Planejada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
