# SPEC-1-001 — Layout, sidebar e navegação aplicada

**Fase:** 1  
**Status:** planejada  
**Dono:** Engenharia de produto (execução) e Thórus (validação do fluxo)  
**Origem no escopo:** seção 4.9; rota `/dashboard e /projetos (shell)`  
**Degrau da solução:** construção mínima da tela e da regra local necessárias para esta operação funcionar manualmente.

## Contexto e decisões fechadas

- **Estado atual:** não existe aplicativo executável neste repositório; a SPEC antiga correspondente está registrada no mapa de migração, commit `a359cdc`.
- **Estado desejado:** uma pessoa com o papel indicado completa a operação em `/dashboard e /projetos (shell)` e encontra o resultado persistido, autorizado e auditável.
- **Decisões já fechadas:** Fase 1 é manual; dados próprios ficam no banco C2; auth e grants C3/C4; UI D1–D5; nenhum conector de negócio externo nesta fase.
- **Bloqueios/dependências:** B1 para código. Aprovação visual do piloto usa a base neutra de D2; a ausência de marca oficial não bloqueia desenho navegável. Dependências técnicas de domínio: D1–D5 e C2–C4.

## Perguntas de validação do cliente

| Perguntas | O que confirmar / como responder | Referência |
|---|---|---|
| P1–P2: projetos piloto, campos e fonte de verdade; usar Passos A–E para conferir Asana sem editar. | Responder marcando opções ou com uma frase curta. Se não souber, indicar quem confirma. Para Asana, anotar nomes/valores e usar somente GET; não alterar dados nem enviar token. | [Questionário P1–P10](../../03_documentos/06-Questionario-levantamento-cliente.md) |

## Resultado observável

Navegar entre Dashboard/Projetos e aba do projeto entregue mantendo URL e foco.

## Limites e dependências

- **Inclui:** rota `/dashboard e /projetos (shell)`, leitura/mutação desta SPEC, persistência pertinente, grants de servidor, estados D3, activity C3 e evidência abaixo.
- **Fora de escopo:** Construir todas as páginas num único lote ou conectar serviço externo.
- **Entradas e pré-condições:** C2–C5, UI D1–D5, grants descritos abaixo; B1 para código. Aprovação visual do piloto usa a base neutra de D2; a ausência de marca oficial não bloqueia desenho navegável.
- **Saídas/artefatos:** página funcional, response envelope C3, migration/constraint necessários, eventos de auditoria, evidências CA.
- **Dependências e responsáveis:** dependências listadas no cabeçalho; Produto/Engenharia fecha B1; Thórus fecha perguntas P referenciadas.
- **Atores e permissões mínimas:** Identidade local determina itens disponíveis; Configurações somente Admin. Abas dependem de read do recurso.
- **Superfícies/arquivos/configurações afetadas:** rota, componentes próprios, endpoint/handler, tabelas C2 citadas, autorização C4 e audit event.
- **Risco e plano B:** ocultação visual não protege dados; aplicar a mesma decisão no servidor. Pendência de vocabulário usa somente fixture identificada até resposta P.
- **Rollback ou reversão:** tela pode suspender ação sem apagar histórico; migration aditiva/reversível; cancelamento preserva registro e emite auditoria.

## Página e comportamento detalhado

URL/rota: `/dashboard e /projetos (shell)`. Base visual única: [Contrato de interface](01-contrato-interface.md).

| Área | Conteúdo e interação |
|---|---|
| Sidebar | Aplicar D1/D2; item ativo por rota; expandir Configurações; recolher pelo botão acessível |
| Header | Breadcrumb e título da página; slots para filtros/CTA, sem inventar uma barra global de busca |
| Projeto | Header comum e seis links de abas; título/código vem do GET autorizado, não da URL |
| Página de referência | Renderizar lista real de capacidade do usuário, estado carregando/negado e componente de formulário local de demonstração |
| Mobile | Drawer menu fecha ao navegar, devolve foco ao botão; cards substituem tabela nas rotas entregues |

## Dados de entrada e saída

| Entrada | Regra |
|---|---|
| me.name/role/capabilities | Somente resposta autenticada C4; sem controlar permissão por string exibida |
| route/returnTo | Caminho interno allowlisted; active item calculado da URL |
| projectHeader | id/code/name/client/archived; carregar apenas após autorização |

## Dados e integrações

UI chama somente API local C3; tabelas respeitam FKs/índices C2. Nenhuma API Asana/Drive é chamada nesta rota.

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Erro |
|---|---|---|---|---|---|
| Tela `/dashboard e /projetos (shell)` ↔ API local ↔ banco | Banco da aplicação | Campos listados em Dados de entrada; versão/C2 | Identidade local determina itens disponíveis; Configurações somente Admin. Abas dependem de read do recurso. | C3: paginação, Idempotency-Key em escrita, expectedVersion em edição; sem retry automático de PATCH | Envelope C3, manter rascunho, correlationId |
| API local ↔ activity_events | Banco na mesma transação | ator, recurso, ação, alvo, instante UTC e resumo seguro | Herda grant da ação | Transação atômica; chave impede repetição de evento | Se log falhar, cancelar mutação e devolver erro C3 |

### Regras de negócio e regras de dados

D1–D5 é fonte de verdade do design; todas as SPECs reutilizam os componentes entregues aqui.
Menus das rotas não entregues ficam desabilitados somente no protótipo de navegação; produção inclui apenas rotas funcionais. Route guard continua no servidor.
Guardar preferência de sidebar localmente é permitido; identidade/dados do projeto não ficam no localStorage.

## API local desta entrega

- GET /api/me → nome, perfil, policyVersion, capabilities; contrato consumido da SPEC-1-002.
- GET /api/projects/{id}/header → código/nome/cliente/arquivado; consumo da SPEC-1-008. Shell permite fixture na demonstração até essas rotas existirem.

## Fluxo principal e caminhos de erro

1. Construir uma página de referência com componentes D2 e estados D3 em 375/768/1440 px.
2. Entrar no shell com fixture Admin e CS; testar expansão de menu e breadcrumb.
3. Navegar entre Dashboard/Projetos e aba do projeto entregue mantendo URL e foco.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Construir uma página de referência com componentes D2 e estados D3 em 375/768/1440 px. | Admin e CS recebem menu correspondente; largura/foco/área de toque seguem D1/D2. | Erro de gravação mantém formulário segundo D3. |
| Permissão/limite | Identidade local determina itens disponíveis; Configurações somente Admin. Abas dependem de read do recurso. | Links e voltar/avançar mantêm rota ativa e breadcrumb; keyboard abre e fecha menu. | Responder 401/403/404 conforme C3, sem vazamento. |
| Falha/concorrência | A página de referência mostra carregamento, vazio, erro e formulário inválido sem perder valores. | Dados permanecem íntegros, sem sucesso falso. | 409/422/503 recuperável; campos mantidos e nenhuma alteração parcial. |

## Critérios de aceite

- [ ] **SPEC-1-001-CA-01:** Admin e CS recebem menu correspondente; largura/foco/área de toque seguem D1/D2.
- [ ] **SPEC-1-001-CA-02:** Links e voltar/avançar mantêm rota ativa e breadcrumb; keyboard abre e fecha menu.
- [ ] **SPEC-1-001-CA-03:** A página de referência mostra carregamento, vazio, erro e formulário inválido sem perder valores.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | SPEC-1-001-CA-01 e SPEC-1-001-CA-03 com fixture C5 | Antes da implementação, executar a ação de UI/API descrita nesta SPEC para os critérios principal, negado e conflito | Falha por ausência da regra/comportamento, sem falha de fixture; registrar status/estado observado | Saída do cenário e estado inicial fictício |
| GREEN | Todos os CA desta SPEC | Implementar a rota ponta a ponta; repetir UI/API, reler do banco e tentar request sem grant/com versão obsoleta | Cada CA passa; mutação e evento são atômicos; erro mantém rascunho | Captura por estado, resposta C3 redigida, releitura e evento |
| REFACTOR | Todos os CA desta SPEC | Reexecutar fluxo, teclado e viewports 375/768/1440; aplicar o runner escolhido em B1 quando houver código | Sem regressão; critérios continuam binários; autorização server-side e dados íntegros | Relatório focal e evidências por CA |

**Dados/fixtures:** [Contrato de dados, C5]; adicionar somente as entidades exigidas pela tabela Dados de entrada desta SPEC.  
**Caminhos de erro obrigatórios:** Navegar entre Dashboard/Projetos e aba do projeto entregue mantendo URL e foco. Erro de permissão, referência inválida, conflito de versão e falha de banco conforme C3.  
**Evidência exigida:** um artefato por critério; captura de página com dado fictício, API response sem segredo, verificação de persistência/autorização.

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** [contrato visual](01-contrato-interface.md), [dados/API](02-contrato-dados-api.md), seção de limite/permissão desta SPEC e C2/C3/C4.
2. **Alterar somente:** rota `/dashboard e /projetos (shell)`, componente, handler/endpoint e persistência deste fluxo.
3. **Não alterar:** Construir todas as páginas num único lote ou conectar serviço externo.
4. **Executar nesta ordem:** migration/contrato; leitura autorizada; ação principal e commit; estado de erro; teclado/mobile; captura de evidência.
5. **Parar e pedir validação quando:** B1 para código. Aprovação visual do piloto usa a base neutra de D2; a ausência de marca oficial não bloqueia desenho navegável.
6. **Estado válido ao parar:** rota principal não grava parcial; sessão/permissões protegem resposta; dados existentes e histórico continuam íntegros.

## Checklist de execução

- [ ] Grant e lista de campos confirmados; fixture mínima tem chave/teste repetível.
- [ ] Caminho principal liga tela, servidor, banco e evento de auditoria na mesma entrega.
- [ ] Vazio, erro, inválido, negado, concorrência e sucesso aplicam D3 na rota.
- [ ] Teclado/foco/viewport e evidências por CA verificados.
- [ ] Thórus confere o fluxo com dados fictícios; pendência de produto registrada.

## Handoff e operação

- **Como demonstrar:** Navegar entre Dashboard/Projetos e aba do projeto entregue mantendo URL e foco.
- **Como operar depois:** Thórus opera pela rota indicada usando os grants C4.
- **Como monitorar:** correlationId, falha de gravação, 401/403, conflito e evento da rota; nenhum log guarda segredo.
- **Pendência conhecida:** B1 para código. Aprovação visual do piloto usa a base neutra de D2; a ausência de marca oficial não bloqueia desenho navegável.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Subseção exata | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| F1-05 | Construir sidebar, breadcrumb e rota de shell | Thorus | SPEC-1-001 | Admin e CS veem somente navegação autorizada; menu/voltar/avançar mantêm rota e foco. | Página e comportamento detalhado; D1 | SPEC-1-001-CA-01; cenário de UI/API, releitura e evidência | Roteiro de navegação com captura 1440/768/375 e teclado. | D1/D2 e rota local disponível. | Parar se ação principal depender de hover ou página for apenas botão sem destino funcional. | Planejada |
| F1-16 | Aplicar design system, estados base, teclado e responsividade | Thorus | SPEC-1-001 | Shell em 375/768/1440 px usa tokens/componentes e estados base, acessível por teclado. | D2–D4 e Critérios de aceite | SPEC-1-001-CA-02; cenário de UI/API, releitura e evidência | Matriz viewport/foco/contraste, screenshots e erros de formulário vinculados. | D1–D5 aceitos internamente e rota de shell. | Parar se foco, label, contraste, zoom ou manutenção de rascunho falhar. | Planejada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
