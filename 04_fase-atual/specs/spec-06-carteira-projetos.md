# SPEC-1-006 — Carteira: busca, filtros e paginação

**Fase:** 1  
**Status:** planejada  
**Dono:** Engenharia de produto (execução) e Thórus (validação do fluxo)  
**Origem no escopo:** seção 4.1 e 4.9; rota `/projetos`  
**Degrau da solução:** construção mínima da tela e da regra local necessárias para esta operação funcionar manualmente.

## Contexto e decisões fechadas

- **Estado atual:** não existe aplicativo executável neste repositório; a SPEC antiga correspondente está registrada no mapa de migração, commit `a359cdc`.
- **Estado desejado:** uma pessoa com o papel indicado completa a operação em `/projetos` e encontra o resultado persistido, autorizado e auditável.
- **Decisões já fechadas:** Fase 1 é manual; dados próprios ficam no banco C2; auth e grants C3/C4; UI D1–D5; nenhum conector de negócio externo nesta fase.
- **Bloqueios/dependências:** B1 para implementação de código; B3 para autenticação real. P8 para liberar usuários reais. Dependências técnicas de domínio: SPEC-1-001, SPEC-1-002, SPEC-1-004, SPEC-1-007

## Perguntas de validação do cliente

| Perguntas | O que confirmar / como responder | Referência |
|---|---|---|
| P1, P2 e P5: projetos piloto, campos, status e opções atuais no Asana (Passos A–D). | Responder marcando opções ou com uma frase curta. Se não souber, indicar quem confirma. Para Asana, anotar nomes/valores e usar somente GET; não alterar dados nem enviar token. | [Questionário P1–P10](../../03_documentos/06-Questionario-levantamento-cliente.md) |

## Resultado observável

Limpa filtros, usa arquivados e compara com Liderança autorizada a todos.

## Limites e dependências

- **Inclui:** rota `/projetos`, leitura/mutação desta SPEC, persistência pertinente, grants de servidor, estados D3, activity C3 e evidência abaixo.
- **Fora de escopo:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
- **Entradas e pré-condições:** C2–C5, UI D1–D5, grants descritos abaixo; B1 para implementação de código; B3 para autenticação real. P8 para liberar usuários reais.
- **Saídas/artefatos:** página funcional, response envelope C3, migration/constraint necessários, eventos de auditoria, evidências CA.
- **Dependências e responsáveis:** dependências listadas no cabeçalho; Produto/Engenharia fecha B1; Thórus fecha perguntas P referenciadas.
- **Atores e permissões mínimas:** projects.read por escopo; Novo/Editar/Arquivar só quando capability da ação permitir.
- **Superfícies/arquivos/configurações afetadas:** rota, componentes próprios, endpoint/handler, tabelas C2 citadas, autorização C4 e audit event.
- **Risco e plano B:** ocultação visual não protege dados; aplicar a mesma decisão no servidor. Pendência de vocabulário usa somente fixture identificada até resposta P.
- **Rollback ou reversão:** tela pode suspender ação sem apagar histórico; migration aditiva/reversível; cancelamento preserva registro e emite auditoria.

## Página e comportamento detalhado

URL/rota: `/projetos`. Base visual única: [Contrato de interface](01-contrato-interface.md).

| Área | Conteúdo e interação |
|---|---|
| Header | Projetos, contador autorizado, CTA Novo projeto → /projetos/novo |
| Filtros | Busca código/nome/cliente/empreendimento; cliente, fase, situação, CS, responsável técnico, UF, ativos/arquivados; Limpar filtros |
| Tabela | Código/Nome, Cliente, Município/UF, Fase, Situação, CS, Responsável técnico, Próximo prazo, Atualizado em; clicar nome abre Visão geral |
| Paginação | 25/50 itens, página/total, ordenação nome/atualizado/próximo prazo; query string preservada |
| Ações por linha | Abrir, Editar, Arquivar/Reativar; modal e efeito em SPEC-1-008 |
| Mobile | Cards com nome, cliente, situação, prazo, responsável e menu; D4 |

## Dados de entrada e saída

| Parâmetro | Regra |
|---|---|
| q | trim, máximo 120; busca normalizada; debounce 300 ms; Enter consulta imediatamente |
| client/phase/status/cs/technical/state/archived | IDs/enum allowlisted; combinar com AND; arquivados excluídos por padrão |
| next_due_date | Mínima data não nula entre tarefas não concluídas do projeto; sem tarefa → Não informado |
| sort/page/pageSize | C3; next_due asc deixa sem prazo ao final; empate por id |

## Dados e integrações

UI chama somente API local C3; tabelas respeitam FKs/índices C2. Nenhuma API Asana/Drive é chamada nesta rota.

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Erro |
|---|---|---|---|---|---|
| Tela `/projetos` ↔ API local ↔ banco | Banco da aplicação | Campos listados em Dados de entrada; versão/C2 | projects.read por escopo; Novo/Editar/Arquivar só quando capability da ação permitir. | C3: paginação, Idempotency-Key em escrita, expectedVersion em edição; sem retry automático de PATCH | Envelope C3, manter rascunho, correlationId |
| API local ↔ activity_events | Banco na mesma transação | ator, recurso, ação, alvo, instante UTC e resumo seguro | Herda grant da ação | Transação atômica; chave impede repetição de evento | Se log falhar, cancelar mutação e devolver erro C3 |

### Regras de negócio e regras de dados

Filtrar por autorização antes de agregação/busca; nenhum resultado oculto entra no total.
Requisição de busca mais antiga não substitui resultado recente; cancelar/ignorar respostas fora de ordem.
Após voltar do detalhe, recuperar query/página; após arquivo, atualizar listagem e manter filtros.
Cliente no filtro só aparece se possui algum projeto visível ou se usuário tem gestão global; não listar clientes de carteira invisível.

## API local desta entrega

- GET /api/projects?q=&client=&phase=&status=&cs=&technical=&state=&archived=&sort=&page=&pageSize= → registros e total autorizados.
- Consumir ações de arquivo da SPEC-1-008 e formulário SPEC-1-007; não duplicar mutações nesta rota.

## Fluxo principal e caminhos de erro

1. CS abre carteira de A, busca código e combina fase + responsável.
2. Abre projeto e volta: filtros/página se mantêm.
3. Limpa filtros, usa arquivados e compara com Liderança autorizada a todos.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | CS abre carteira de A, busca código e combina fase + responsável. | Consulta, total, filtros e próxima data representam apenas os projetos autorizados. | Erro de gravação mantém formulário segundo D3. |
| Permissão/limite | projects.read por escopo; Novo/Editar/Arquivar só quando capability da ação permitir. | Busca rápida não exibe resposta obsoleta e URL preserva paginação ao retornar. | Responder 401/403/404 conforme C3, sem vazamento. |
| Falha/concorrência | Base vazia, filtro vazio, loading, erro, acesso negado e mobile têm estados concretos. | Dados permanecem íntegros, sem sucesso falso. | 409/422/503 recuperável; campos mantidos e nenhuma alteração parcial. |

## Critérios de aceite

- [ ] **SPEC-1-006-CA-01:** Consulta, total, filtros e próxima data representam apenas os projetos autorizados.
- [ ] **SPEC-1-006-CA-02:** Busca rápida não exibe resposta obsoleta e URL preserva paginação ao retornar.
- [ ] **SPEC-1-006-CA-03:** Base vazia, filtro vazio, loading, erro, acesso negado e mobile têm estados concretos.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | SPEC-1-006-CA-01 e SPEC-1-006-CA-03 com fixture C5 | Antes da implementação, executar a ação de UI/API descrita nesta SPEC para os critérios principal, negado e conflito | Falha por ausência da regra/comportamento, sem falha de fixture; registrar status/estado observado | Saída do cenário e estado inicial fictício |
| GREEN | Todos os CA desta SPEC | Implementar a rota ponta a ponta; repetir UI/API, reler do banco e tentar request sem grant/com versão obsoleta | Cada CA passa; mutação e evento são atômicos; erro mantém rascunho | Captura por estado, resposta C3 redigida, releitura e evento |
| REFACTOR | Todos os CA desta SPEC | Reexecutar fluxo, teclado e viewports 375/768/1440; aplicar o runner escolhido em B1 quando houver código | Sem regressão; critérios continuam binários; autorização server-side e dados íntegros | Relatório focal e evidências por CA |

**Dados/fixtures:** [Contrato de dados, C5]; adicionar somente as entidades exigidas pela tabela Dados de entrada desta SPEC.  
**Caminhos de erro obrigatórios:** Limpa filtros, usa arquivados e compara com Liderança autorizada a todos. Erro de permissão, referência inválida, conflito de versão e falha de banco conforme C3.  
**Evidência exigida:** um artefato por critério; captura de página com dado fictício, API response sem segredo, verificação de persistência/autorização.

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** [contrato visual](01-contrato-interface.md), [dados/API](02-contrato-dados-api.md), seção de limite/permissão desta SPEC e SPEC-1-001, SPEC-1-002, SPEC-1-004, SPEC-1-007.
2. **Alterar somente:** rota `/projetos`, componente, handler/endpoint e persistência deste fluxo.
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

- **Como demonstrar:** Limpa filtros, usa arquivados e compara com Liderança autorizada a todos.
- **Como operar depois:** Thórus opera pela rota indicada usando os grants C4.
- **Como monitorar:** correlationId, falha de gravação, 401/403, conflito e evento da rota; nenhum log guarda segredo.
- **Pendência conhecida:** B1 para implementação de código; B3 para autenticação real. P8 para liberar usuários reais.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Subseção exata | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| F1-06 | Implementar carteira com busca, filtros e paginação autorizados | Thorus | SPEC-1-006 | Lista, total, busca e filtros incluem exatamente projetos acessíveis ao usuário. | Página e comportamento detalhado; API local | SPEC-1-006-CA-01; cenário de UI/API, releitura e evidência | Capturas + requests autorizados/negados + comparação de resultados 25/50. | Dados C5 e read projects; SPEC-1-007/008. | Parar se busca, total, filtro ou agregação expor projeto sem grant. | Planejada |
| F1-22 | Validar filtros, privacidade e estados de retorno da carteira | Thorus | SPEC-1-006 | Filtro autorizado e paginação continuam corretos ao voltar; projeto B nunca aparece em resultado/total. | API local; Estados e erros | SPEC-1-006-CA-02; cenário de UI/API, releitura e evidência | Matriz query × perfil, retorno browser e captura sem resultados. | F1-06 e C5. | Parar diante de resultado obsoleto ou vazamento por filtro/agregação. | Planejada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
