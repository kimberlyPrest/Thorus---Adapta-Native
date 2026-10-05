# SPEC-1-013 — Aba Documentos: referências manuais e organização

**Fase:** 1  
**Status:** planejada  
**Dono:** Engenharia de produto (execução) e Thórus (validação do fluxo)  
**Origem no escopo:** seção 4.3 e 4.9; rota `/projetos/{id}/documentos`  
**Degrau da solução:** construção mínima da tela e da regra local necessárias para esta operação funcionar manualmente.

## Contexto e decisões fechadas

- **Estado atual:** não existe aplicativo executável neste repositório; a SPEC antiga correspondente está registrada no mapa de migração, commit `a359cdc`.
- **Estado desejado:** uma pessoa com o papel indicado completa a operação em `/projetos/{id}/documentos` e encontra o resultado persistido, autorizado e auditável.
- **Decisões já fechadas:** Fase 1 é manual; dados próprios ficam no banco C2; auth e grants C3/C4; UI D1–D5; nenhum conector de negócio externo nesta fase.
- **Bloqueios/dependências:** P3 organiza referência documental futura; não bloqueia link manual no MVP. P8 governa visibilidade interna. Dependências técnicas de domínio: SPEC-1-001, SPEC-1-002, SPEC-1-004, SPEC-1-005, SPEC-1-008

## Perguntas de validação do cliente

| Perguntas | O que confirmar / como responder | Referência |
|---|---|---|
| P3–P4 e P8: modelo de pastas, notas Gemini, link/importação e compartilhamento. | Responder marcando opções ou com uma frase curta. Se não souber, indicar quem confirma. Para Asana, anotar nomes/valores e usar somente GET; não alterar dados nem enviar token. | [Questionário P1–P10](../../03_documentos/06-Questionario-levantamento-cliente.md) |

## Resultado observável

Editar nome ou arquivar sem afetar documento de origem.

## Limites e dependências

- **Inclui:** rota `/projetos/{id}/documentos`, leitura/mutação desta SPEC, persistência pertinente, grants de servidor, estados D3, activity C3 e evidência abaixo.
- **Fora de escopo:** Integrações Asana/Drive/Gemini/WhatsApp; criar/alterar compartilhamento ou enviar automaticamente. Atendimento manual é detalhado na SPEC-1-018.
- **Entradas e pré-condições:** C2–C5, UI D1–D5, grants descritos abaixo; P3 organiza referência documental futura; não bloqueia link manual no MVP. P8 governa visibilidade interna.
- **Saídas/artefatos:** página funcional, response envelope C3, migration/constraint necessários, eventos de auditoria, evidências CA.
- **Dependências e responsáveis:** dependências listadas no cabeçalho; Produto/Engenharia fecha B1; Thórus fecha perguntas P referenciadas.
- **Atores e permissões mínimas:** documents.read/create/update/archive por projeto. Todas as referências são internas nesta fase; não controlar autorização Drive pelo cadastro.
- **Superfícies/arquivos/configurações afetadas:** rota, componentes próprios, endpoint/handler, tabelas C2 citadas, autorização C4 e audit event.
- **Risco e plano B:** ocultação visual não protege dados; aplicar a mesma decisão no servidor. Pendência de vocabulário usa somente fixture identificada até resposta P.
- **Rollback ou reversão:** tela pode suspender ação sem apagar histórico; migration aditiva/reversível; cancelamento preserva registro e emite auditoria.

## Página e comportamento detalhado

URL/rota: `/projetos/{id}/documentos`. Base visual única: [Contrato de interface](01-contrato-interface.md).

| Área | Conteúdo e interação |
|---|---|
| Lista | Nome, Categoria, Domínio do link, Adicionado por/em, Ativo/Arquivado, Ações; busca/filtro |
| Adicionar/editar | Drawer Nome, Categoria, URL e Observação; label “Link externo informado manualmente” |
| Abrir | Nova aba por clique, link mostra domínio e ícone de externo; rel noopener/noreferrer |
| Arquivar | Confirmação preserva registro/autoria; filtro Arquivados permite reler |
| Sem documentos | “Nenhum link cadastrado”; CTA Adicionar referência quando permitido |

## Dados de entrada e saída

| Campo | Regra |
|---|---|
| name/url | Obrigatórios C2/C3; URL HTTP(S), sem credenciais/controle |
| category/observation | Opcionais; categoria ativa para novo; texto não executa HTML |
| origin | manual imutável; não interpretar domínio como autorização ou sync |
| visibility | Interna herdada do projeto; sem seletor “Público/Cliente” nesta fase |

## Dados e integrações

UI chama somente API local C3; tabelas respeitam FKs/índices C2. Nenhuma API Asana/Drive é chamada nesta rota.

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Erro |
|---|---|---|---|---|---|
| Tela `/projetos/{id}/documentos` ↔ API local ↔ banco | Banco da aplicação | Campos listados em Dados de entrada; versão/C2 | documents.read/create/update/archive por projeto. Todas as referências são internas nesta fase; não controlar autorização Drive pelo cadastro. | C3: paginação, Idempotency-Key em escrita, expectedVersion em edição; sem retry automático de PATCH | Envelope C3, manter rascunho, correlationId |
| API local ↔ activity_events | Banco na mesma transação | ator, recurso, ação, alvo, instante UTC e resumo seguro | Herda grant da ação | Transação atômica; chave impede repetição de evento | Se log falhar, cancelar mutação e devolver erro C3 |

### Regras de negócio e regras de dados

Não buscar URL no servidor, baixar arquivo, gerar thumbnail nem usar HEAD para validar acesso.
Abrir endereço externo é ação manual do navegador; descrever claramente se a pessoa recebe negação no destino.
URL igual pode ser cadastrada duas vezes com nomes diferentes por intenção; UI avisa referência similar no mesmo projeto, sem merge automático.
Arquivar referência nunca remove arquivo externo. A referência não comprova autorização de envio; na SPEC-1-018, operador confirma ACL no Drive para o destinatário antes do envio manual.

## API local desta entrega

- GET /api/projects/{id}/documents?q=&category=&archived=&page= → updated_at desc/id.
- POST /api/projects/{id}/documents {name,categoryId,url,observation}; PATCH /documents/{id} {campos,expectedVersion}.
- POST /documents/{id}/archive ou /restore {expectedVersion} → registro + evento atômicos; nenhuma requisição Drive.

## Fluxo principal e caminhos de erro

1. Adicionar referência com nome/categoria/link e salvar.
2. Abrir link manualmente e reler registro após nova sessão.
3. Editar nome ou arquivar sem afetar documento de origem.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Adicionar referência com nome/categoria/link e salvar. | Referência aparece persistida com nome/categoria/domínio/autoria e abre por clique. | Erro de gravação mantém formulário segundo D3. |
| Permissão/limite | documents.read/create/update/archive por projeto. Todas as referências são internas nesta fase; não controlar autorização Drive pelo cadastro. | Editar/arquivo respeitam versionamento/permite leitura histórica. | Responder 401/403/404 conforme C3, sem vazamento. |
| Falha/concorrência | URL perigosa/ inválida, projeto invisível e falha são tratados; salvar/listar não inicia tráfego ao serviço externo. | Dados permanecem íntegros, sem sucesso falso. | 409/422/503 recuperável; campos mantidos e nenhuma alteração parcial. |

## Critérios de aceite

- [ ] **SPEC-1-013-CA-01:** Referência aparece persistida com nome/categoria/domínio/autoria e abre por clique.
- [ ] **SPEC-1-013-CA-02:** Editar/arquivo respeitam versionamento/permite leitura histórica.
- [ ] **SPEC-1-013-CA-03:** URL perigosa/ inválida, projeto invisível e falha são tratados; salvar/listar não inicia tráfego ao serviço externo.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | SPEC-1-013-CA-01 e SPEC-1-013-CA-03 com fixture C5 | Antes da implementação, executar a ação de UI/API descrita nesta SPEC para os critérios principal, negado e conflito | Falha por ausência da regra/comportamento, sem falha de fixture; registrar status/estado observado | Saída do cenário e estado inicial fictício |
| GREEN | Todos os CA desta SPEC | Implementar a rota ponta a ponta; repetir UI/API, reler do banco e tentar request sem grant/com versão obsoleta | Cada CA passa; mutação e evento são atômicos; erro mantém rascunho | Captura por estado, resposta C3 redigida, releitura e evento |
| REFACTOR | Todos os CA desta SPEC | Reexecutar fluxo, teclado e viewports 375/768/1440; aplicar o runner escolhido em B1 quando houver código | Sem regressão; critérios continuam binários; autorização server-side e dados íntegros | Relatório focal e evidências por CA |

**Dados/fixtures:** [Contrato de dados, C5]; adicionar somente as entidades exigidas pela tabela Dados de entrada desta SPEC.  
**Caminhos de erro obrigatórios:** Editar nome ou arquivar sem afetar documento de origem. Erro de permissão, referência inválida, conflito de versão e falha de banco conforme C3.  
**Evidência exigida:** um artefato por critério; captura de página com dado fictício, API response sem segredo, verificação de persistência/autorização.

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** [contrato visual](01-contrato-interface.md), [dados/API](02-contrato-dados-api.md), seção de limite/permissão desta SPEC e SPEC-1-001, SPEC-1-002, SPEC-1-004, SPEC-1-005, SPEC-1-008.
2. **Alterar somente:** rota `/projetos/{id}/documentos`, componente, handler/endpoint e persistência deste fluxo.
3. **Não alterar:** Integrações Asana/Drive/Gemini/WhatsApp e comunicação externa.
4. **Executar nesta ordem:** migration/contrato; leitura autorizada; ação principal e commit; estado de erro; teclado/mobile; captura de evidência.
5. **Parar e pedir validação quando:** P3 organiza referência documental futura; não bloqueia link manual no MVP. P8 governa visibilidade interna.
6. **Estado válido ao parar:** rota principal não grava parcial; sessão/permissões protegem resposta; dados existentes e histórico continuam íntegros.

## Checklist de execução

- [ ] Grant e lista de campos confirmados; fixture mínima tem chave/teste repetível.
- [ ] Caminho principal liga tela, servidor, banco e evento de auditoria na mesma entrega.
- [ ] Vazio, erro, inválido, negado, concorrência e sucesso aplicam D3 na rota.
- [ ] Teclado/foco/viewport e evidências por CA verificados.
- [ ] Thórus confere o fluxo com dados fictícios; pendência de produto registrada.

## Handoff e operação

- **Como demonstrar:** Editar nome ou arquivar sem afetar documento de origem.
- **Como operar depois:** Thórus opera pela rota indicada usando os grants C4.
- **Como monitorar:** correlationId, falha de gravação, 401/403, conflito e evento da rota; nenhum log guarda segredo.
- **Pendência conhecida:** P3 organiza referência documental futura; não bloqueia link manual no MVP. P8 governa visibilidade interna.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Subseção exata | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| F1-13 | Implementar cadastro, edição e arquivo de referências documentais | Thorus | SPEC-1-013 | Referência manual persiste e é lida; arquivo/URL original permanece intocado pela aplicação. | Página e comportamento detalhado; API local | SPEC-1-013-CA-01; cenário de UI/API, releitura e evidência | Captura da lista/drawer, link manual, auditoria e teste de URL rejeitada. | P3/P8 e grants documents. | Parar se caminho exigir upload, criação de pasta, ACL ou leitura automática do Drive. | Planejada |
| F1-29 | Validar URL e proteger falhas/arquivamento de referência | Thorus | SPEC-1-013 | URL inválida/insegura é bloqueada; erro mantém campos; arquivar preserva documento externo e referência histórica. | Regras de negócio; D3 estados | SPEC-1-013-CA-02; cenário de UI/API, releitura e evidência | Casos javascript/data/HTTP inválida + TLS URL manual segura; arquivado. | F1-13, C3 e P3/P8. | Parar se backend fizer fetch, preview ou remover o arquivo externo. | Planejada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
