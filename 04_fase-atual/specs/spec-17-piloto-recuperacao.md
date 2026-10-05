# SPEC-1-017 — Piloto manual, persistência e recuperação comprovados

**Fase:** 1  
**Status:** planejada  
**Dono:** Engenharia de produto (execução) e Thórus (validação do fluxo)  
**Origem no escopo:** seção 1, 7 e 11; rota `Roteiro das rotas entregues + ambiente isolado de backup`  
**Degrau da solução:** construção mínima da tela e da regra local necessárias para esta operação funcionar manualmente.

## Contexto e decisões fechadas

- **Estado atual:** não existe aplicativo executável neste repositório; a SPEC antiga correspondente está registrada no mapa de migração, commit `a359cdc`.
- **Estado desejado:** uma pessoa com o papel indicado completa a operação em `Roteiro das rotas entregues + ambiente isolado de backup` e encontra o resultado persistido, autorizado e auditável.
- **Decisões já fechadas:** Fase 1 é manual; dados próprios ficam no banco C2; auth e grants C3/C4; UI D1–D5; nenhum conector de negócio externo nesta fase.
- **Bloqueios/dependências:** Todas rotas implementadas e B1/B3 resolvidos; política de retenção/backup P8 documentada antes de uso com dados reais. Dependências técnicas de domínio: SPEC-1-001, SPEC-1-002, SPEC-1-003, SPEC-1-004, SPEC-1-005, SPEC-1-006, SPEC-1-007, SPEC-1-008, SPEC-1-009, SPEC-1-010, SPEC-1-011, SPEC-1-012, SPEC-1-013, SPEC-1-014, SPEC-1-015, SPEC-1-016

## Perguntas de validação do cliente

| Perguntas | O que confirmar / como responder | Referência |
|---|---|---|
| P1 e P10: amostra autorizada para piloto e participantes da medição. | Responder marcando opções ou com uma frase curta. Se não souber, indicar quem confirma. Para Asana, anotar nomes/valores e usar somente GET; não alterar dados nem enviar token. | [Questionário P1–P10](../../03_documentos/06-Questionario-levantamento-cliente.md) |

## Resultado observável

Backup/restaurar no ambiente isolado e comparar relações/versões/contagens; colher aceite humano da fase.

## Limites e dependências

- **Inclui:** rota `Roteiro das rotas entregues + ambiente isolado de backup`, leitura/mutação desta SPEC, persistência pertinente, grants de servidor, estados D3, activity C3 e evidência abaixo.
- **Fora de escopo:** Deploy de produção, restore sobre banco real ou execução de conectores Fase 2.
- **Entradas e pré-condições:** C2–C5, UI D1–D5, grants descritos abaixo; Todas rotas implementadas e B1/B3 resolvidos; política de retenção/backup P8 documentada antes de uso com dados reais.
- **Saídas/artefatos:** página funcional, response envelope C3, migration/constraint necessários, eventos de auditoria, evidências CA.
- **Dependências e responsáveis:** dependências listadas no cabeçalho; Produto/Engenharia fecha B1; Thórus fecha perguntas P referenciadas.
- **Atores e permissões mínimas:** CS/Engenharia/Legais testam suas rotas; Admin + Engenharia operam backup em ambiente isolado com acesso registrado.
- **Superfícies/arquivos/configurações afetadas:** rota, componentes próprios, endpoint/handler, tabelas C2 citadas, autorização C4 e audit event.
- **Risco e plano B:** ocultação visual não protege dados; aplicar a mesma decisão no servidor. Pendência de vocabulário usa somente fixture identificada até resposta P.
- **Rollback ou reversão:** tela pode suspender ação sem apagar histórico; migration aditiva/reversível; cancelamento preserva registro e emite auditoria.

## Página e comportamento detalhado

URL/rota: `Roteiro das rotas entregues + ambiente isolado de backup`. Base visual única: [Contrato de interface](01-contrato-interface.md).

| Superfície | Procedimento |
|---|---|
| Piloto | Admin cria usuário/matriz/listas; CS cadastra projeto; equipe percorre todas abas e Dashboard |
| Evidências | Captura por rota/estado com dado fictício; releitura depois de nova sessão; relatório de permissão concedida/negada |
| Restore | Backup pelo mecanismo B1 identificado, hash/instante/versão; restaurar em banco novo de teste; mesma aplicação aponta somente para teste |
| Comparação | Contagens/FKs, autorias, versões vigentes e arquivamentos antes/depois; checklists das SPECs de origem |
| Falha | Reabrir task/SPEC de origem; registrar causa e evidência; não declarar fase aceita se dado/UX falhar |

## Dados de entrada e saída

| Dado | Regra |
|---|---|
| fixture set | C5 com vínculos, versões, pedidos, tarefas, eventos, referências e histórico |
| backup metadata | timestamp UTC, schema version, checksum, responsável; segredo fora do relatório |
| baseline P10 | Cronometrar localizar status/última definição/documento, antes/depois; amostra mínima 3 projetos representativos |
| evidence | Um link de artefato por CA; ausência de implementação ainda não é resultado de teste aprovado |

## Dados e integrações

UI chama somente API local C3; tabelas respeitam FKs/índices C2. Nenhuma API Asana/Drive é chamada nesta rota.

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Erro |
|---|---|---|---|---|---|
| Tela `Roteiro das rotas entregues + ambiente isolado de backup` ↔ API local ↔ banco | Banco da aplicação | Campos listados em Dados de entrada; versão/C2 | CS/Engenharia/Legais testam suas rotas; Admin + Engenharia operam backup em ambiente isolado com acesso registrado. | C3: paginação, Idempotency-Key em escrita, expectedVersion em edição; sem retry automático de PATCH | Envelope C3, manter rascunho, correlationId |
| API local ↔ activity_events | Banco na mesma transação | ator, recurso, ação, alvo, instante UTC e resumo seguro | Herda grant da ação | Transação atômica; chave impede repetição de evento | Se log falhar, cancelar mutação e devolver erro C3 |

### Regras de negócio e regras de dados

Restaurar somente banco novo isolado; comparar snapshot e relações, sem sobrescrever ambiente real.
Aceite exige todas rotas principais, estados de erro, permissões e versão vigentes corretos; página bonita sem gravação não atende.
Integrações de negócio zero: distinguir requests ao backend e login B3 de tráfego Asana/Drive.
Escopo de P10 medir ganho após implementação; este documento não inventa resultado alcançado.

## API local desta entrega

- Não há endpoint de restaurar produção nesta fase. Procedimento do runtime B1 tem comando real registrado antes de executar.
- GETs e mutações seguem os contratos das SPECs 002–016; transações/releitura C3 comprovam persistência.
- Desligar ferramentas Asana/Drive/Gemini/WhatsApp na demonstração não deve impedir fluxo manual; autenticação B3 mantém mecanismo escolhido.

## Fluxo principal e caminhos de erro

1. Executar cenário C5: projeto, tarefa, definição v1, solicitação/v2, legal, documento, timeline e dashboard.
2. Sair/entrar, reler entidades e testar conta sem permissão.
3. Backup/restaurar no ambiente isolado e comparar relações/versões/contagens; colher aceite humano da fase.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Executar cenário C5: projeto, tarefa, definição v1, solicitação/v2, legal, documento, timeline e dashboard. | Equipe demonstra cada rota e persiste os dados após nova sessão conforme sua permissão. | Erro de gravação mantém formulário segundo D3. |
| Permissão/limite | CS/Engenharia/Legais testam suas rotas; Admin + Engenharia operam backup em ambiente isolado com acesso registrado. | Restore isolado conserva entidades, FKs, autoria, vigentes e arquivo do snapshot. | Responder 401/403/404 conforme C3, sem vazamento. |
| Falha/concorrência | Todas evidências e P10 baseline/pós ficam registrados; falhas remetem à SPEC de origem antes do aceite. | Dados permanecem íntegros, sem sucesso falso. | 409/422/503 recuperável; campos mantidos e nenhuma alteração parcial. |

## Critérios de aceite

- [ ] **SPEC-1-017-CA-01:** Equipe demonstra cada rota e persiste os dados após nova sessão conforme sua permissão.
- [ ] **SPEC-1-017-CA-02:** Restore isolado conserva entidades, FKs, autoria, vigentes e arquivo do snapshot.
- [ ] **SPEC-1-017-CA-03:** Todas evidências e P10 baseline/pós ficam registrados; falhas remetem à SPEC de origem antes do aceite.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | SPEC-1-017-CA-01 e SPEC-1-017-CA-03 com fixture C5 | Antes da implementação, executar a ação de UI/API descrita nesta SPEC para os critérios principal, negado e conflito | Falha por ausência da regra/comportamento, sem falha de fixture; registrar status/estado observado | Saída do cenário e estado inicial fictício |
| GREEN | Todos os CA desta SPEC | Implementar a rota ponta a ponta; repetir UI/API, reler do banco e tentar request sem grant/com versão obsoleta | Cada CA passa; mutação e evento são atômicos; erro mantém rascunho | Captura por estado, resposta C3 redigida, releitura e evento |
| REFACTOR | Todos os CA desta SPEC | Reexecutar fluxo, teclado e viewports 375/768/1440; aplicar o runner escolhido em B1 quando houver código | Sem regressão; critérios continuam binários; autorização server-side e dados íntegros | Relatório focal e evidências por CA |

**Dados/fixtures:** [Contrato de dados, C5]; adicionar somente as entidades exigidas pela tabela Dados de entrada desta SPEC.  
**Caminhos de erro obrigatórios:** Backup/restaurar no ambiente isolado e comparar relações/versões/contagens; colher aceite humano da fase. Erro de permissão, referência inválida, conflito de versão e falha de banco conforme C3.  
**Evidência exigida:** um artefato por critério; captura de página com dado fictício, API response sem segredo, verificação de persistência/autorização.

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** [contrato visual](01-contrato-interface.md), [dados/API](02-contrato-dados-api.md), seção de limite/permissão desta SPEC e SPEC-1-001, SPEC-1-002, SPEC-1-003, SPEC-1-004, SPEC-1-005, SPEC-1-006, SPEC-1-007, SPEC-1-008, SPEC-1-009, SPEC-1-010, SPEC-1-011, SPEC-1-012, SPEC-1-013, SPEC-1-014, SPEC-1-015, SPEC-1-016.
2. **Alterar somente:** rota `Roteiro das rotas entregues + ambiente isolado de backup`, componente, handler/endpoint e persistência deste fluxo.
3. **Não alterar:** Deploy de produção, restore sobre banco real ou execução de conectores Fase 2.
4. **Executar nesta ordem:** migration/contrato; leitura autorizada; ação principal e commit; estado de erro; teclado/mobile; captura de evidência.
5. **Parar e pedir validação quando:** Todas rotas implementadas e B1/B3 resolvidos; política de retenção/backup P8 documentada antes de uso com dados reais.
6. **Estado válido ao parar:** rota principal não grava parcial; sessão/permissões protegem resposta; dados existentes e histórico continuam íntegros.

## Checklist de execução

- [ ] Grant e lista de campos confirmados; fixture mínima tem chave/teste repetível.
- [ ] Caminho principal liga tela, servidor, banco e evento de auditoria na mesma entrega.
- [ ] Vazio, erro, inválido, negado, concorrência e sucesso aplicam D3 na rota.
- [ ] Teclado/foco/viewport e evidências por CA verificados.
- [ ] Thórus confere o fluxo com dados fictícios; pendência de produto registrada.

## Handoff e operação

- **Como demonstrar:** Backup/restaurar no ambiente isolado e comparar relações/versões/contagens; colher aceite humano da fase.
- **Como operar depois:** Thórus opera pela rota indicada usando os grants C4.
- **Como monitorar:** correlationId, falha de gravação, 401/403, conflito e evento da rota; nenhum log guarda segredo.
- **Pendência conhecida:** Todas rotas implementadas e B1/B3 resolvidos; política de retenção/backup P8 documentada antes de uso com dados reais.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Subseção exata | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| F1-10 | Demonstrar o percurso manual completo e cópia/restauração isolada | Thorus | SPEC-1-017 | Projeto fictício conclui o fluxo; restore em banco isolado conserva dados, vínculos, versões e autoria. | Página e comportamento detalhado; Fluxo principal e erros | SPEC-1-017-CA-01; cenário de UI/API, releitura e evidência | Checklist por rota com links, relatório de comparação e aceite CS/Engenharia. | SPECs 001–016 entregues, B1/B3 e política de backup conhecida. | Parar se o destino não estiver isolado, restore perder histórico ou etapa depender de Asana/Drive. | Planejada |
| F1-34 | Validar recuperação isolada de dados e critérios de aceite | Thorus | SPEC-1-017 | Snapshot de backup restaura em banco de teste com contagens, relacionamentos e versões conferidos. | Página e comportamento detalhado; Restore | SPEC-1-017-CA-02; cenário de UI/API, releitura e evidência | Checksum/time/schema e relatório before/after sem dados reais. | B1, política P8, F1-10, todas rotas manuais entregues. | Parar se destino não for banco vazio isolado ou dado/histórico divergir. | Planejada |
| F1-35 | Executar regressão de persistência e consistência ponta a ponta | Thorus | SPEC-1-017 | C5 percorre usuários, permissões, projeto, abas, dashboard, logout e reread sem chamadas de negócio externas. | Fluxo principal e erros; Restore | SPEC-1-017-CA-03; cenário de UI/API, releitura e evidência | Links para resultados por CA, captura por rota, requisições/respostas e P10. | Toda entrega SPEC-1-001 a SPEC-1-016 fechada. | Parar em qualquer aceite vermelho; reabrir task e SPEC da rota causadora. | Planejada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
