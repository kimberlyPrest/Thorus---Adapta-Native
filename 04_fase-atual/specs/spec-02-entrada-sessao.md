# SPEC-1-002 — Entrada, sessão e saída do usuário

**Fase:** 1  
**Status:** planejada  
**Dono:** Engenharia de produto (execução) e Thórus (validação do fluxo)  
**Origem no escopo:** seção 3 e 4.9; rota `/entrar`  
**Degrau da solução:** construção mínima da tela e da regra local necessárias para esta operação funcionar manualmente.

## Contexto e decisões fechadas

- **Estado atual:** não existe aplicativo executável neste repositório; a SPEC antiga correspondente está registrada no mapa de migração, commit `a359cdc`.
- **Estado desejado:** uma pessoa com o papel indicado completa a operação em `/entrar` e encontra o resultado persistido, autorizado e auditável.
- **Decisões já fechadas:** Fase 1 é manual; dados próprios ficam no banco C2; auth e grants C3/C4; UI D1–D5; nenhum conector de negócio externo nesta fase.
- **Bloqueios/dependências:** B1/B3 precisam ser decididos antes de implementar sessão. UI de Entrar e cenários de fixture podem ser desenhados imediatamente. Dependências técnicas de domínio: SPEC-1-001

## Perguntas de validação do cliente

| Perguntas | O que confirmar / como responder | Referência |
|---|---|---|
| P8: acesso por perfil e portal; não compartilhar credenciais. | Responder marcando opções ou com uma frase curta. Se não souber, indicar quem confirma. Para Asana, anotar nomes/valores e usar somente GET; não alterar dados nem enviar token. | [Questionário P1–P10](../../03_documentos/06-Questionario-levantamento-cliente.md) |

## Resultado observável

Sair revoga sessão; tentar GET anterior exige autenticação.

## Limites e dependências

- **Inclui:** rota `/entrar`, leitura/mutação desta SPEC, persistência pertinente, grants de servidor, estados D3, activity C3 e evidência abaixo.
- **Fora de escopo:** Criar provedor de identidade próprio, cadastro público, OAuth Asana ou convites por e-mail.
- **Entradas e pré-condições:** C2–C5, UI D1–D5, grants descritos abaixo; B1/B3 precisam ser decididos antes de implementar sessão. UI de Entrar e cenários de fixture podem ser desenhados imediatamente.
- **Saídas/artefatos:** página funcional, response envelope C3, migration/constraint necessários, eventos de auditoria, evidências CA.
- **Dependências e responsáveis:** dependências listadas no cabeçalho; Produto/Engenharia fecha B1; Thórus fecha perguntas P referenciadas.
- **Atores e permissões mínimas:** Rota Entrar pública; dados internos exigem sessão e conta local ativa/ready. Criar conta local é tarefa da SPEC-1-003.
- **Superfícies/arquivos/configurações afetadas:** rota, componentes próprios, endpoint/handler, tabelas C2 citadas, autorização C4 e audit event.
- **Risco e plano B:** ocultação visual não protege dados; aplicar a mesma decisão no servidor. Pendência de vocabulário usa somente fixture identificada até resposta P.
- **Rollback ou reversão:** tela pode suspender ação sem apagar histórico; migration aditiva/reversível; cancelamento preserva registro e emite auditoria.

## Página e comportamento detalhado

URL/rota: `/entrar`. Base visual única: [Contrato de interface](01-contrato-interface.md).

| Área | Conteúdo e interação |
|---|---|
| Card central | Nome do produto, “Acesse com sua conta corporativa”, CTA do provedor definido em B3; largura 420 px |
| Sessão expirada | Texto específico e continuar para returnTo interno após autenticar |
| Cadastro pendente | “Seu acesso ainda não foi liberado. Contate o administrador”; não mostrar projetos |
| Erro de login | Orientação Tentar novamente sem stack trace; CTA retorna ao fluxo autenticador |
| Sair | Ação no footer da sidebar; concluir revogação e voltar a Entrar |

## Dados de entrada e saída

| Entrada | Regra |
|---|---|
| identity_subject | ID estável verificado pelo provedor B3; e-mail informado isoladamente nunca autentica |
| local user | Subject vinculado + active=true + access_state=ready |
| returnTo | Somente rota interna; default /dashboard; negar redirect externo |

## Dados e integrações

UI chama somente API local C3; tabelas respeitam FKs/índices C2. Nenhuma API Asana/Drive é chamada nesta rota.

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Erro |
|---|---|---|---|---|---|
| Tela `/entrar` ↔ API local ↔ banco | Banco da aplicação | Campos listados em Dados de entrada; versão/C2 | Rota Entrar pública; dados internos exigem sessão e conta local ativa/ready. Criar conta local é tarefa da SPEC-1-003. | C3: paginação, Idempotency-Key em escrita, expectedVersion em edição; sem retry automático de PATCH | Envelope C3, manter rascunho, correlationId |
| API local ↔ activity_events | Banco na mesma transação | ator, recurso, ação, alvo, instante UTC e resumo seguro | Herda grant da ação | Transação atômica; chave impede repetição de evento | Se log falhar, cancelar mutação e devolver erro C3 |

### Regras de negócio e regras de dados

Não armazenar senha corporativa nem usar token Asana como autenticação.
Revogação local/conta desativada é aplicada na próxima requisição; login/logout e negações são auditados sem credenciais.
Sessão usa mecanismo seguro suportado pelo runtime B1; se cookie, Secure/HttpOnly/SameSite e CSRF no endpoint mutável; fechar configuração antes de GREEN.

## API local desta entrega

- Adaptar início/callback ao provedor de B3, com validação de state/nonce conforme provedor; configuração concreta registrada na task de preparação.
- GET /api/me → id,name,role,policyVersion,capabilities. Sem sessão: 401; conta inativa: revogar e 401.
- POST /api/session/logout → revogar sessão corrente; 200 sem payload de projeto.

## Fluxo principal e caminhos de erro

1. Selecionar e registrar B3, callback e expiração suportada; preparar contas fixture.
2. Usuário autorizado autentica e abre returnTo interno; usuário inexistente/inativo recebe mensagem sem dados.
3. Sair revoga sessão; tentar GET anterior exige autenticação.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Selecionar e registrar B3, callback e expiração suportada; preparar contas fixture. | Conta ativa/vinculada entra; desconhecida/inativa não retorna dados. | Erro de gravação mantém formulário segundo D3. |
| Permissão/limite | Rota Entrar pública; dados internos exigem sessão e conta local ativa/ready. Criar conta local é tarefa da SPEC-1-003. | Sessão expirada volta a Entrar e retoma só URL interna após login. | Responder 401/403/404 conforme C3, sem vazamento. |
| Falha/concorrência | Logout e desativação negam requisição seguinte e registram auditoria segura. | Dados permanecem íntegros, sem sucesso falso. | 409/422/503 recuperável; campos mantidos e nenhuma alteração parcial. |

## Critérios de aceite

- [ ] **SPEC-1-002-CA-01:** Conta ativa/vinculada entra; desconhecida/inativa não retorna dados.
- [ ] **SPEC-1-002-CA-02:** Sessão expirada volta a Entrar e retoma só URL interna após login.
- [ ] **SPEC-1-002-CA-03:** Logout e desativação negam requisição seguinte e registram auditoria segura.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | SPEC-1-002-CA-01 e SPEC-1-002-CA-03 com fixture C5 | Antes da implementação, executar a ação de UI/API descrita nesta SPEC para os critérios principal, negado e conflito | Falha por ausência da regra/comportamento, sem falha de fixture; registrar status/estado observado | Saída do cenário e estado inicial fictício |
| GREEN | Todos os CA desta SPEC | Implementar a rota ponta a ponta; repetir UI/API, reler do banco e tentar request sem grant/com versão obsoleta | Cada CA passa; mutação e evento são atômicos; erro mantém rascunho | Captura por estado, resposta C3 redigida, releitura e evento |
| REFACTOR | Todos os CA desta SPEC | Reexecutar fluxo, teclado e viewports 375/768/1440; aplicar o runner escolhido em B1 quando houver código | Sem regressão; critérios continuam binários; autorização server-side e dados íntegros | Relatório focal e evidências por CA |

**Dados/fixtures:** [Contrato de dados, C5]; adicionar somente as entidades exigidas pela tabela Dados de entrada desta SPEC.  
**Caminhos de erro obrigatórios:** Sair revoga sessão; tentar GET anterior exige autenticação. Erro de permissão, referência inválida, conflito de versão e falha de banco conforme C3.  
**Evidência exigida:** um artefato por critério; captura de página com dado fictício, API response sem segredo, verificação de persistência/autorização.

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** [contrato visual](01-contrato-interface.md), [dados/API](02-contrato-dados-api.md), seção de limite/permissão desta SPEC e SPEC-1-001.
2. **Alterar somente:** rota `/entrar`, componente, handler/endpoint e persistência deste fluxo.
3. **Não alterar:** Criar provedor de identidade próprio, cadastro público, OAuth Asana ou convites por e-mail.
4. **Executar nesta ordem:** migration/contrato; leitura autorizada; ação principal e commit; estado de erro; teclado/mobile; captura de evidência.
5. **Parar e pedir validação quando:** B1/B3 precisam ser decididos antes de implementar sessão. UI de Entrar e cenários de fixture podem ser desenhados imediatamente.
6. **Estado válido ao parar:** rota principal não grava parcial; sessão/permissões protegem resposta; dados existentes e histórico continuam íntegros.

## Checklist de execução

- [ ] Grant e lista de campos confirmados; fixture mínima tem chave/teste repetível.
- [ ] Caminho principal liga tela, servidor, banco e evento de auditoria na mesma entrega.
- [ ] Vazio, erro, inválido, negado, concorrência e sucesso aplicam D3 na rota.
- [ ] Teclado/foco/viewport e evidências por CA verificados.
- [ ] Thórus confere o fluxo com dados fictícios; pendência de produto registrada.

## Handoff e operação

- **Como demonstrar:** Sair revoga sessão; tentar GET anterior exige autenticação.
- **Como operar depois:** Thórus opera pela rota indicada usando os grants C4.
- **Como monitorar:** correlationId, falha de gravação, 401/403, conflito e evento da rota; nenhum log guarda segredo.
- **Pendência conhecida:** B1/B3 precisam ser decididos antes de implementar sessão. UI de Entrar e cenários de fixture podem ser desenhados imediatamente.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Subseção exata | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| F1-04 | Implementar entrada, sessão e proteção de rotas | Thorus | SPEC-1-002 | Usuário ativo autenticado chega ao returnTo interno; pessoa sem vínculo/inativa recebe bloqueio sem dados. | UI e fluxos; Contrato de dados e API C3 | SPEC-1-002-CA-01; cenário de UI/API, releitura e evidência | Capturas Entrar + casos 401/403, evento auth redigido e sessão verificada. | B1 e B3 fechados; SPEC-1-001 e fixtures prontos. | Parar se sessão puder ser forjada, revogação não surtir efeito ou segredo aparecer em UI/log. | Planejada |
| F1-17 | Validar expiração, logout e conta inativa no servidor | Thorus | SPEC-1-002 | A sessão deixa de autorizar rota na próxima requisição após logout, expiração ou inativação. | API local; Regras de negócio | SPEC-1-002-CA-02; cenário de UI/API, releitura e evidência | Prova HTTP 401 e tela Entrar com returnTo preservado. | B3/provider/runtime de sessão resolvidos. | Parar sem comportamento de revogação conhecido. | Planejada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
