# SPEC-1-005 — Listas operacionais editáveis pelo Admin

**Fase:** 1  
**Status:** planejada  
**Dono:** Engenharia de produto (execução) e Thórus (validação do fluxo)  
**Origem no escopo:** seção 4.6 e 4.9; rota `/configuracoes/listas`  
**Degrau da solução:** construção mínima da tela e da regra local necessárias para esta operação funcionar manualmente.

## Contexto e decisões fechadas

- **Estado atual:** não existe aplicativo executável neste repositório; a SPEC antiga correspondente está registrada no mapa de migração, commit `a359cdc`.
- **Estado desejado:** uma pessoa com o papel indicado completa a operação em `/configuracoes/listas` e encontra o resultado persistido, autorizado e auditável.
- **Decisões já fechadas:** Fase 1 é manual; dados próprios ficam no banco C2; auth e grants C3/C4; UI D1–D5; nenhum conector de negócio externo nesta fase.
- **Bloqueios/dependências:** P5/P9 antes de registrar fatos reais com vocabulário oficial. Opções demonstrativas não exigem integração. Dependências técnicas de domínio: SPEC-1-001, SPEC-1-002, SPEC-1-004

## Perguntas de validação do cliente

| Perguntas | O que confirmar / como responder | Referência |
|---|---|---|
| P5 e P8: nomes das fases/status, eventos e quem registra/confirma. | Responder marcando opções ou com uma frase curta. Se não souber, indicar quem confirma. Para Asana, anotar nomes/valores e usar somente GET; não alterar dados nem enviar token. | [Questionário P1–P10](../../03_documentos/06-Questionario-levantamento-cliente.md) |

## Resultado observável

Desativar opção a remove de novas seleções; registro antigo continua legível.

## Limites e dependências

- **Inclui:** rota `/configuracoes/listas`, leitura/mutação desta SPEC, persistência pertinente, grants de servidor, estados D3, activity C3 e evidência abaixo.
- **Fora de escopo:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
- **Entradas e pré-condições:** C2–C5, UI D1–D5, grants descritos abaixo; P5/P9 antes de registrar fatos reais com vocabulário oficial. Opções demonstrativas não exigem integração.
- **Saídas/artefatos:** página funcional, response envelope C3, migration/constraint necessários, eventos de auditoria, evidências CA.
- **Dependências e responsáveis:** dependências listadas no cabeçalho; Produto/Engenharia fecha B1; Thórus fecha perguntas P referenciadas.
- **Atores e permissões mínimas:** Admin com admin_lists.read/update. Demais usuários consultam somente valores ativos permitidos para seus formulários.
- **Superfícies/arquivos/configurações afetadas:** rota, componentes próprios, endpoint/handler, tabelas C2 citadas, autorização C4 e audit event.
- **Risco e plano B:** ocultação visual não protege dados; aplicar a mesma decisão no servidor. Pendência de vocabulário usa somente fixture identificada até resposta P.
- **Rollback ou reversão:** tela pode suspender ação sem apagar histórico; migration aditiva/reversível; cancelamento preserva registro e emite auditoria.

## Página e comportamento detalhado

URL/rota: `/configuracoes/listas`. Base visual única: [Contrato de interface](01-contrato-interface.md).

| Área | Conteúdo e interação |
|---|---|
| Navegação local | Fases, Situações, Disciplinas, Prioridades, Tipos legais, Status legais, Categorias de documentos |
| Lista | Label, ordem, ativo/inativo, registros referenciando; busca; CTA “Adicionar opção” |
| Editor | Nome, chave estável proposta e ordem; desativar com confirmação; chave imutável após criação |
| Ordenação | Campo de posição e botões Subir/Descer acionáveis por teclado; salvar ordem inteira de categoria |
| Efeito | Mensagem “Inativa para novos registros; preservada nos anteriores” |

## Dados de entrada e saída

| Campo | Regra |
|---|---|
| kind/key/label/position | C2; kind allowlisted, key estável única na categoria; label obrigatório |
| active | Desativação nunca remove FK ou substitui o valor em registros antigos |
| task state / request state | Não são catálogos editáveis; transições fixadas em SPEC-1-009/011 |

## Dados e integrações

UI chama somente API local C3; tabelas respeitam FKs/índices C2. Nenhuma API Asana/Drive é chamada nesta rota.

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Erro |
|---|---|---|---|---|---|
| Tela `/configuracoes/listas` ↔ API local ↔ banco | Banco da aplicação | Campos listados em Dados de entrada; versão/C2 | Admin com admin_lists.read/update. Demais usuários consultam somente valores ativos permitidos para seus formulários. | C3: paginação, Idempotency-Key em escrita, expectedVersion em edição; sem retry automático de PATCH | Envelope C3, manter rascunho, correlationId |
| API local ↔ activity_events | Banco na mesma transação | ator, recurso, ação, alvo, instante UTC e resumo seguro | Herda grant da ação | Transação atômica; chave impede repetição de evento | Se log falhar, cancelar mutação e devolver erro C3 |

### Regras de negócio e regras de dados

Não permitir lista vazia de tipo/status legal se impedir novos registros; mostrar impacto antes de desativar última opção ativa.
Label atualizado aparece nas consultas; mudanças anteriores continuam descritas pela auditoria. Desativar não renomeia história.
Na demonstração, lista inicial usa opções explicitamente marcadas como base de piloto; P5/P9 validam nomes oficiais.
Não converter estados executáveis de tarefa/pedido em opções livres, para não quebrar máquina de estados.

## API local desta entrega

- GET /api/admin/catalogs/{kind}?active=; POST /api/admin/catalogs/{kind} {key,label,position}.
- PATCH /api/admin/catalogs/{kind}/{id} {label,active,expectedVersion}; PUT /api/admin/catalogs/{kind}/order {ids,expectedVersion}.
- GET /api/catalogs/{kind}?active=true para selects, autorizado de acordo com recurso consumidor.

## Fluxo principal e caminhos de erro

1. Admin escolhe categoria, adiciona opção e salva.
2. Usuário abre formulário da aba correspondente e encontra opção ativa na ordem configurada.
3. Desativar opção a remove de novas seleções; registro antigo continua legível.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Admin escolhe categoria, adiciona opção e salva. | Criar, renomear, reordenar e desativar pelo Admin é persistido e consumido pelos formulários. | Erro de gravação mantém formulário segundo D3. |
| Permissão/limite | Admin com admin_lists.read/update. Demais usuários consultam somente valores ativos permitidos para seus formulários. | Opção inativa já referenciada mantém leitura e não é selecionada para novo registro. | Responder 401/403/404 conforme C3, sem vazamento. |
| Falha/concorrência | Chave duplicada, posição inválida, conflito e última opção necessária exibem validação sem perder dados. | Dados permanecem íntegros, sem sucesso falso. | 409/422/503 recuperável; campos mantidos e nenhuma alteração parcial. |

## Critérios de aceite

- [ ] **SPEC-1-005-CA-01:** Criar, renomear, reordenar e desativar pelo Admin é persistido e consumido pelos formulários.
- [ ] **SPEC-1-005-CA-02:** Opção inativa já referenciada mantém leitura e não é selecionada para novo registro.
- [ ] **SPEC-1-005-CA-03:** Chave duplicada, posição inválida, conflito e última opção necessária exibem validação sem perder dados.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | SPEC-1-005-CA-01 e SPEC-1-005-CA-03 com fixture C5 | Antes da implementação, executar a ação de UI/API descrita nesta SPEC para os critérios principal, negado e conflito | Falha por ausência da regra/comportamento, sem falha de fixture; registrar status/estado observado | Saída do cenário e estado inicial fictício |
| GREEN | Todos os CA desta SPEC | Implementar a rota ponta a ponta; repetir UI/API, reler do banco e tentar request sem grant/com versão obsoleta | Cada CA passa; mutação e evento são atômicos; erro mantém rascunho | Captura por estado, resposta C3 redigida, releitura e evento |
| REFACTOR | Todos os CA desta SPEC | Reexecutar fluxo, teclado e viewports 375/768/1440; aplicar o runner escolhido em B1 quando houver código | Sem regressão; critérios continuam binários; autorização server-side e dados íntegros | Relatório focal e evidências por CA |

**Dados/fixtures:** [Contrato de dados, C5]; adicionar somente as entidades exigidas pela tabela Dados de entrada desta SPEC.  
**Caminhos de erro obrigatórios:** Desativar opção a remove de novas seleções; registro antigo continua legível. Erro de permissão, referência inválida, conflito de versão e falha de banco conforme C3.  
**Evidência exigida:** um artefato por critério; captura de página com dado fictício, API response sem segredo, verificação de persistência/autorização.

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** [contrato visual](01-contrato-interface.md), [dados/API](02-contrato-dados-api.md), seção de limite/permissão desta SPEC e SPEC-1-001, SPEC-1-002, SPEC-1-004.
2. **Alterar somente:** rota `/configuracoes/listas`, componente, handler/endpoint e persistência deste fluxo.
3. **Não alterar:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
4. **Executar nesta ordem:** migration/contrato; leitura autorizada; ação principal e commit; estado de erro; teclado/mobile; captura de evidência.
5. **Parar e pedir validação quando:** P5/P9 antes de registrar fatos reais com vocabulário oficial. Opções demonstrativas não exigem integração.
6. **Estado válido ao parar:** rota principal não grava parcial; sessão/permissões protegem resposta; dados existentes e histórico continuam íntegros.

## Checklist de execução

- [ ] Grant e lista de campos confirmados; fixture mínima tem chave/teste repetível.
- [ ] Caminho principal liga tela, servidor, banco e evento de auditoria na mesma entrega.
- [ ] Vazio, erro, inválido, negado, concorrência e sucesso aplicam D3 na rota.
- [ ] Teclado/foco/viewport e evidências por CA verificados.
- [ ] Thórus confere o fluxo com dados fictícios; pendência de produto registrada.

## Handoff e operação

- **Como demonstrar:** Desativar opção a remove de novas seleções; registro antigo continua legível.
- **Como operar depois:** Thórus opera pela rota indicada usando os grants C4.
- **Como monitorar:** correlationId, falha de gravação, 401/403, conflito e evento da rota; nenhum log guarda segredo.
- **Pendência conhecida:** P5/P9 antes de registrar fatos reais com vocabulário oficial. Opções demonstrativas não exigem integração.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Subseção exata | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| F1-20 | Criar e editar opções de catálogo pela área Configurações | Thorus | SPEC-1-005 | Admin cria, renomeia e ordena uma opção e select de registro lista o valor ativo. | Página e comportamento detalhado | SPEC-1-005-CA-01; cenário de UI/API, releitura e evidência | Capturas Admin e aba consumidora; query options. | P5/P9 e primeira rota consumidora. | Parar se estado executável for transformado em opção arbitrária. | Planejada |
| F1-21 | Desativar opções sem apagar o histórico referenciado | Thorus | SPEC-1-005 | Opção inativa sai de novos selects, permanece legível em registros existentes. | Regras de negócio; Estados e erros | SPEC-1-005-CA-02; cenário de UI/API, releitura e evidência | Demonstração antes/depois e FK preservada. | Catálogo com registro fixture associado. | Parar se exclusão física ou lista essencial vazia ocorrer. | Planejada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
