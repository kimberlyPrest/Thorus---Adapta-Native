# SPEC-1-010 — Aba Definições: vigente e histórico por disciplina

**Fase:** 1  
**Status:** planejada  
**Dono:** Engenharia de produto (execução) e Thórus (validação do fluxo)  
**Origem no escopo:** seção 4.4 e 4.9; rota `/projetos/{id}/definicoes`  
**Degrau da solução:** construção mínima da tela e da regra local necessárias para esta operação funcionar manualmente.

## Contexto e decisões fechadas

- **Estado atual:** não existe aplicativo executável neste repositório; a SPEC antiga correspondente está registrada no mapa de migração, commit `a359cdc`.
- **Estado desejado:** uma pessoa com o papel indicado completa a operação em `/projetos/{id}/definicoes` e encontra o resultado persistido, autorizado e auditável.
- **Decisões já fechadas:** Fase 1 é manual; dados próprios ficam no banco C2; auth e grants C3/C4; UI D1–D5; nenhum conector de negócio externo nesta fase.
- **Bloqueios/dependências:** P9 antes de aprovar definições reais; fixtures usam designação explícita e disciplina demonstrativa. Dependências técnicas de domínio: SPEC-1-001, SPEC-1-002, SPEC-1-004, SPEC-1-005, SPEC-1-008

## Perguntas de validação do cliente

| Perguntas | O que confirmar / como responder | Referência |
|---|---|---|
| P5 e P9: definições técnicas atuais, histórico e quem pode alterá-las. | Responder marcando opções ou com uma frase curta. Se não souber, indicar quem confirma. Para Asana, anotar nomes/valores e usar somente GET; não alterar dados nem enviar token. | [Questionário P1–P10](../../03_documentos/06-Questionario-levantamento-cliente.md) |

## Resultado observável

Seleciona Propor alteração e inicia pedido separado com baseVersion atual.

## Limites e dependências

- **Inclui:** rota `/projetos/{id}/definicoes`, leitura/mutação desta SPEC, persistência pertinente, grants de servidor, estados D3, activity C3 e evidência abaixo.
- **Fora de escopo:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
- **Entradas e pré-condições:** C2–C5, UI D1–D5, grants descritos abaixo; P9 antes de aprovar definições reais; fixtures usam designação explícita e disciplina demonstrativa.
- **Saídas/artefatos:** página funcional, response envelope C3, migration/constraint necessários, eventos de auditoria, evidências CA.
- **Dependências e responsáveis:** dependências listadas no cabeçalho; Produto/Engenharia fecha B1; Thórus fecha perguntas P referenciadas.
- **Atores e permissões mínimas:** definitions.read; registrar inicial exige definitions.register_initial + designação de aprovador na disciplina. Propor abre fluxo SPEC-1-011.
- **Superfícies/arquivos/configurações afetadas:** rota, componentes próprios, endpoint/handler, tabelas C2 citadas, autorização C4 e audit event.
- **Risco e plano B:** ocultação visual não protege dados; aplicar a mesma decisão no servidor. Pendência de vocabulário usa somente fixture identificada até resposta P.
- **Rollback ou reversão:** tela pode suspender ação sem apagar histórico; migration aditiva/reversível; cancelamento preserva registro e emite auditoria.

## Página e comportamento detalhado

URL/rota: `/projetos/{id}/definicoes`. Base visual única: [Contrato de interface](01-contrato-interface.md).

| Área | Conteúdo e interação |
|---|---|
| Navegação interna | Vigentes / Solicitações; contadores limitados ao read autorizado |
| Disciplina | Filtro/chips por disciplina e pesquisa chave/título |
| Vigente | Card/tabela Título, Valor vigente, Versão, Aprovador, Decidido em; botão Propor alteração |
| Inicial | Drawer Disciplina, Chave estável, Título, Valor, Observação; confirmar “Registrar definição inicial” |
| Histórico | Drawer por definição lista vN, data/ator/motivo e comparação em texto lado a lado; versões imutáveis |
| Sem definição | “Nenhuma definição registrada nesta disciplina”; CTA inicial só ao aprovador elegível |

## Dados de entrada e saída

| Campo | Regra |
|---|---|
| discipline/key/label | Obrigatórios C2; disciplina ativa; chave única normalizada no projeto/disciplina |
| value/observation | Valor inicial 1–4000, observação opcional ≤2000; texto simples |
| currentVersion | Número/valor/decisor/hora da única vigente por current_version_id |
| previousVersion | Imutável; sem API editar diretamente valor de versão |

## Dados e integrações

UI chama somente API local C3; tabelas respeitam FKs/índices C2. Nenhuma API Asana/Drive é chamada nesta rota.

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Erro |
|---|---|---|---|---|---|
| Tela `/projetos/{id}/definicoes` ↔ API local ↔ banco | Banco da aplicação | Campos listados em Dados de entrada; versão/C2 | definitions.read; registrar inicial exige definitions.register_initial + designação de aprovador na disciplina. Propor abre fluxo SPEC-1-011. | C3: paginação, Idempotency-Key em escrita, expectedVersion em edição; sem retry automático de PATCH | Envelope C3, manter rascunho, correlationId |
| API local ↔ activity_events | Banco na mesma transação | ator, recurso, ação, alvo, instante UTC e resumo seguro | Herda grant da ação | Transação atômica; chave impede repetição de evento | Se log falhar, cancelar mutação e devolver erro C3 |

### Regras de negócio e regras de dados

Primeiro registro técnico é confirmação humana pelo aprovador designado; demais registram solicitação, não vigente.
UNIQUE(project,discipline,key) protege registro duplicado; preencher chave não consulta portal externo.
Histórico completo requer definitions.read e vínculo do projeto; log genérico não contém valor técnico integral.
Vigente posterior só muda por decisão da SPEC-1-011; texto vigente não é textarea editável livre.

## API local desta entrega

- GET /api/projects/{id}/definitions?discipline=&q= → vigentes autorizadas.
- POST /api/projects/{id}/definitions {disciplineId,key,label,value,observation} → definição + v1 + current pointer + auditoria atômicos.
- GET /definitions/{definitionId}/versions → histórico ordenado number desc; sem endpoint de overwrite vigente.

## Fluxo principal e caminhos de erro

1. Aprovador cadastra definição inicial em disciplina autorizada.
2. Engenharia consulta vigente e histórico em detalhe.
3. Seleciona Propor alteração e inicia pedido separado com baseVersion atual.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Aprovador cadastra definição inicial em disciplina autorizada. | Registro inicial gera v1 única e é restrito ao aprovador designado. | Erro de gravação mantém formulário segundo D3. |
| Permissão/limite | definitions.read; registrar inicial exige definitions.register_initial + designação de aprovador na disciplina. Propor abre fluxo SPEC-1-011. | Leitor encontra vigente, versões e comparação sem editar histórico. | Responder 401/403/404 conforme C3, sem vazamento. |
| Falha/concorrência | Chave repetida, disciplina inativa, falta de designação e projeto invisível não criam dado parcial. | Dados permanecem íntegros, sem sucesso falso. | 409/422/503 recuperável; campos mantidos e nenhuma alteração parcial. |

## Critérios de aceite

- [ ] **SPEC-1-010-CA-01:** Registro inicial gera v1 única e é restrito ao aprovador designado.
- [ ] **SPEC-1-010-CA-02:** Leitor encontra vigente, versões e comparação sem editar histórico.
- [ ] **SPEC-1-010-CA-03:** Chave repetida, disciplina inativa, falta de designação e projeto invisível não criam dado parcial.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | SPEC-1-010-CA-01 e SPEC-1-010-CA-03 com fixture C5 | Antes da implementação, executar a ação de UI/API descrita nesta SPEC para os critérios principal, negado e conflito | Falha por ausência da regra/comportamento, sem falha de fixture; registrar status/estado observado | Saída do cenário e estado inicial fictício |
| GREEN | Todos os CA desta SPEC | Implementar a rota ponta a ponta; repetir UI/API, reler do banco e tentar request sem grant/com versão obsoleta | Cada CA passa; mutação e evento são atômicos; erro mantém rascunho | Captura por estado, resposta C3 redigida, releitura e evento |
| REFACTOR | Todos os CA desta SPEC | Reexecutar fluxo, teclado e viewports 375/768/1440; aplicar o runner escolhido em B1 quando houver código | Sem regressão; critérios continuam binários; autorização server-side e dados íntegros | Relatório focal e evidências por CA |

**Dados/fixtures:** [Contrato de dados, C5]; adicionar somente as entidades exigidas pela tabela Dados de entrada desta SPEC.  
**Caminhos de erro obrigatórios:** Seleciona Propor alteração e inicia pedido separado com baseVersion atual. Erro de permissão, referência inválida, conflito de versão e falha de banco conforme C3.  
**Evidência exigida:** um artefato por critério; captura de página com dado fictício, API response sem segredo, verificação de persistência/autorização.

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** [contrato visual](01-contrato-interface.md), [dados/API](02-contrato-dados-api.md), seção de limite/permissão desta SPEC e SPEC-1-001, SPEC-1-002, SPEC-1-004, SPEC-1-005, SPEC-1-008.
2. **Alterar somente:** rota `/projetos/{id}/definicoes`, componente, handler/endpoint e persistência deste fluxo.
3. **Não alterar:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
4. **Executar nesta ordem:** migration/contrato; leitura autorizada; ação principal e commit; estado de erro; teclado/mobile; captura de evidência.
5. **Parar e pedir validação quando:** P9 antes de aprovar definições reais; fixtures usam designação explícita e disciplina demonstrativa.
6. **Estado válido ao parar:** rota principal não grava parcial; sessão/permissões protegem resposta; dados existentes e histórico continuam íntegros.

## Checklist de execução

- [ ] Grant e lista de campos confirmados; fixture mínima tem chave/teste repetível.
- [ ] Caminho principal liga tela, servidor, banco e evento de auditoria na mesma entrega.
- [ ] Vazio, erro, inválido, negado, concorrência e sucesso aplicam D3 na rota.
- [ ] Teclado/foco/viewport e evidências por CA verificados.
- [ ] Thórus confere o fluxo com dados fictícios; pendência de produto registrada.

## Handoff e operação

- **Como demonstrar:** Seleciona Propor alteração e inicia pedido separado com baseVersion atual.
- **Como operar depois:** Thórus opera pela rota indicada usando os grants C4.
- **Como monitorar:** correlationId, falha de gravação, 401/403, conflito e evento da rota; nenhum log guarda segredo.
- **Pendência conhecida:** P9 antes de aprovar definições reais; fixtures usam designação explícita e disciplina demonstrativa.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Subseção exata | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| F1-26 | Registrar definição inicial e consultar versão/histórico | Thorus | SPEC-1-010 | Designado cria v1 única; leitura exibe vigente e compara histórico imutável. | Página e comportamento detalhado | SPEC-1-010-CA-01; cenário de UI/API, releitura e evidência | Registro definition/v1 e captura Vigentes/Histórico. | P9, F1-24 e catalogs discipline. | Parar se editar valor vigente direto ou duplicar chave versionada. | Planejada |
| F1-36 | Exibir vigentes sem edição indevida e resolver chave/concorrência | Thorus | SPEC-1-010 | Papel sem designação não registra inicial; duas tentativas da mesma chave não criam duas definições. | Regras de negócio; Estados e erros | SPEC-1-010-CA-02; cenário de UI/API, releitura e evidência | 403/409 e leitura histórico/vigente pela interface. | F1-26, C2/C4 e P9. | Parar se detalhe técnico não estiver isolado por projeto/perfil. | Planejada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
