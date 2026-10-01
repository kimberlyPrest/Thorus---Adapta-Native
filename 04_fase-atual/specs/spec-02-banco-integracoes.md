# SPEC F1.2 — Banco de dados e preparação das integrações

**Fase:** 1<br>
**Status:** planejada<br>
**Dono:** Engenharia de produto<br>
**Origem no escopo:** seções 4.1, 4.9 e 6; P1/P2<br>
**Degrau da solução:** construção mínima de persistência relacional própria e contratos desacoplados; sem conectar provedores externos.

## Banco e integridade

Implementar o modelo lógico definido em [visão geral Fase 1](00-visao-geral-fase-1.md), incluindo usuários/papéis, clientes, projetos/membros, fases e status, tarefas, definições/versionamento, eventos legais, referências documentais, auditoria, vínculos externos e mapeamentos. Aplicar migrações versionadas e reversíveis sempre que possível, FKs, constraints, índices por filtros usados na carteira e política de arquivamento.

Regras mínimas:

- Projeto possui ID interno estável; código interno é único quando preenchido, no escopo definido pela Thórus.
- Registro manual armazena `source_type=manual` e ator. O modelo também aceita futuro `source_type=asana|drive|other`, sem criar conector ativo.
- Campos `external_system`, `external_workspace_id`, `external_resource_type`, `external_gid` e `external_permalink` são opcionais. GID é único dentro da combinação sistema/workspace/tipo, quando preenchido.
- Mapeamento liga recurso/campo externo a campo interno, transformação versionada e estado de revisão. Campo sem aprovação não sobrescreve nem preenche silenciosamente um dado manual.
- Versões de definição e eventos de auditoria são preservados. Correção é novo evento/versão quando altera decisão relevante.
- Data/hora persistida em UTC; interface converte para `America/Sao_Paulo`.
- Queries de leitura/escrita sempre consideram autorização do usuário; FK/ID opaco não substitui controle de acesso.
- Segredos, access tokens, refresh tokens, API keys ou payloads sensíveis não são armazenados no escopo de preparação.

## Contrato preparado para Asana e Drive

Definir interfaces internas desacopladas para conectores futuros, sem implementação de rede, por exemplo:

- `ExternalProjectReader.preview(selection, mappingVersion)` e `ExternalProjectReader.import(selection, mappingVersion)` para projetos/tarefas Asana na Fase 2.
- `ExternalDocumentStore.listReferences(project)` e `ExternalDocumentStore.upload(reference, content)` para Drive quando o escopo/ACL for aprovado na Fase 2.
- Resultado de conector deve prever `success|partial|failed`, correlation ID, contagens, erros tipados, `source_updated_at` e `synced_at`.
- Regra de reconciliação será upsert por GID; conflito entre registro manual e fonte externa vai para revisão, nunca para sobrescrita silenciosa.
- Botões/rotas/feature flags de conexão permanecem desabilitados e não fazem chamadas externas na Fase 1. A UI de Configurações pode mostrar “Integração planejada para Fase 2” e os campos de mapeamento aprováveis, sem pedir token.
- Links externos são texto/URL cadastrados manualmente. Só apresentar permalink fornecido/verificado; nunca inferir URL a partir de ID.

## Operação do banco

- Migração precisa poder ser aplicada a banco vazio e preservar registros em atualização de versão.
- Criar dados demonstrativos via seed distinguível de registros reais e removível sem afetar outros dados.
- Exportação controlada deve respeitar permissão e registrar auditoria; backup/restore faz parte do aceite da Fase 1.
- Falha de gravação mostra erro acionável e não apresenta sucesso fictício. Operações com várias tabelas precisam ser transacionais quando consistência exigir.
- Paginação e filtros da carteira são executados no servidor para o tamanho de dados projetado; não carregar toda a base para filtrar no navegador.

## Critérios de aceite

- [ ] **CA-F1.2-01:** Migrações constroem o banco vazio com todas as FKs, constraints e índices previstos.
- [ ] **CA-F1.2-02:** Criar/editar projeto, tarefa, definição, evento legal, referência de documento e usuário grava no banco e continua visível após novo login.
- [ ] **CA-F1.2-03:** Um `external_gid` nulo é permitido; repetição de GID no mesmo sistema/workspace/recurso é rejeitada com mensagem compreensível.
- [ ] **CA-F1.2-04:** Conflito de campo entre dado manual e valor externo simulado é apresentado como pendência de reconciliação; não sobrescreve o manual.
- [ ] **CA-F1.2-05:** Nenhuma requisição DNS/HTTP para Asana/Drive é gerada pelas telas e fluxos da Fase 1.
- [ ] **CA-F1.2-06:** Backup restaurado mantém relacionamentos, autoria e histórico conforme política aprovada.

## Perguntas de validação do cliente

Consulte P1 e P2 no [questionário de levantamento](../../03_documentos/06-Questionario-levantamento-cliente.md). O passo a passo de localizar workspace, projetos, campos, tarefas, status e obter autorização Asana prepara a integração da Fase 2; token não deve ser enviado por formulário ou mensagem.


## Contexto e decisões fechadas

- **Estado atual:** dados de projeto ficam em ferramentas externas; o MVP precisa operar sem elas.
- **Estado desejado:** banco relacional persistente para todo CRUD manual e contratos externos versionáveis que não façam chamadas de rede.
- **Decisões já fechadas:** ID interno é primário; origem manual explícita; external GID opcional e único por sistema/workspace/tipo; FKs e constraints; timestamps UTC; conflito futuro vai para revisão.
- **Bloqueios:** engine/hosting do banco e stack da aplicação devem ser confirmados com o repositório/arquitetura antes da migração física; não escolher fornecedor por suposição.

## Resultado observável

Usuário cria um projeto manual, sai e retorna e encontra projeto, tarefas, definições/versionamento, eventos legais, links documentais e auditoria; nenhum serviço externo é requisitado.

## Limites e dependências

- **Inclui:** esquema lógico, constraints, migrações, índices, seeds isolados, persistência e interfaces de conector inativas.
- **Fora de escopo:** API real Asana/Drive, armazenamento de tokens, importação e escrita em serviços externos.
- **Entradas e pré-condições:** decisão do banco da aplicação; entidades/tipos do modelo em SPEC F1.0; convenções de IDs e retenção.
- **Saídas/artefatos:** migrações; dicionário físico alinhado ao modelo; seeds de demonstração; interfaces externas não conectadas.
- **Atores/permissões:** engenharia implementa; Admin mantém listas externas à decisão técnica; todos os acessos obedecem F1.1.
- **Superfícies afetadas:** schema/migrations, repositories/services, seed e Configurações > Integrações planejadas.
- **Risco/plano B:** se engine escolhida não suportar constraint desejada, parar e propor alternativa antes de afrouxar integridade.
- **Rollback:** snapshot antes de migração; migrações aditivas ou reversíveis; não remover colunas/dados em downgrade destrutivo.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Interface → banco local | Banco do MVP para registros manuais | users/roles, clients, projects/memberships, phases/statuses, tasks, technical_definitions/versions, legal_events, document_references, activity_events, external_links, integration_mappings | Autorização F1.1; credencial de banco no backend | Transação para mudanças relacionadas; retry não pode duplicar criação | Constraint/validação retorna erro acionável e mantém formulário |
| Adapter preparado → Asana/Drive | Nenhuma fonte conectada na Fase 1 | Interface preview/import/list/upload declarada; GID, source_type, mapping_version, correlation_id | Sem token nem conector habilitado | Chamadas não implementadas/desabilitadas | Qualquer tentativa externa não é executada nesta fase |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-F1.2-01 | External GID ausente | Registro manual é válido e usa ID interno | Nenhuma | Modelo desta SPEC |
| RN-F1.2-02 | Sistema/workspace/tipo/GID repetido | Rejeitar duplicata | GID vazio permitido | Integridade externa |
| RN-F1.2-03 | Futuro campo Asana conflita com valor manual | Criar conflito para revisão, não sobrescrever | Somente regra posterior aprovada pode mudar precedência | P2 |

## Fluxo e regras

1. Aplicar migrações ao banco vazio e validar FKs/índices/constraints.
2. Criar seed marcado como demonstração, sem misturar com cliente real.
3. Cada gravação valida entrada e papel, persiste transacionalmente quando necessário e retorna ID estável.
4. Export/backup respeita permissão e inclui relações/histórico; restore é verificado em cópia de teste.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Projeto e registros relacionados criados manualmente | Persistem e são relidos em nova sessão | Mostrar sucesso só após commit |
| Limite | GID externo nulo e depois único válido | Ambos os registros válidos; conflito duplicado rejeitado | Mensagem aponta sistema/workspace/tipo duplicado |
| Falha | Migração ou gravação falha | Dados existentes preservados, sem sucesso fictício | Reverter migração/snapshot; manter erro acionável |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** modelo de entidades da SPEC F1.0, arquitetura real da aplicação e decisão da engine.
2. **Alterar somente:** schema, migrations, camada de persistência, seed e contratos inativos.
3. **Não alterar:** fonte externa, papéis de F1.1, regras técnicas não decididas ou dados reais.
4. **Executar nesta ordem:** selecionar engine suportada; migrar vazio; validar constraints; implementar persistência; validar restore.
5. **Parar quando:** stack/engine ainda não aprovada, migração exigir perda, campo de origem conflitar ou integração gerar tráfego real.
6. **Estado válido ao parar:** migração não destrutiva e banco manual utilizável; conectores desligados.

## Checklist de execução

- [ ] Engine alinhada à arquitetura da aplicação.
- [ ] Modelo e FKs/migrações aplicados em banco vazio.
- [ ] CRUD manual relido após nova sessão.
- [ ] GID opcional/unicidade/conflito verificados.
- [ ] Backup e restore em ambiente de teste evidenciados.
- [ ] Nenhuma chamada externa observada.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | Gravação de projeto/tarefa não persistida e GID duplicado aceito indevidamente | Testes de integração do repository/constraints contra banco de teste | Falha antes da migração e constraint | Relatório migration/integration |
| GREEN | Migrações, CRUD, FKs, GID único e conflito manual simulado | Rodar migration e suíte focada de persistência | Persistência funciona; duplicata é rejeitada; conflito não sobrescreve | Relatório de execução e fixture redigida |
| REFACTOR/REGRESSÃO | Nova instalação vazia, atualização, seed, restore e desligamento externo | Aplicar migrations em banco limpo e cópia; observar tráfego de rede | Dados íntegros em ciclo manual; zero chamadas externas | Evidência do restore e log de rede |

**Dados/fixtures:** cliente/projeto fictícios, dois usuários, tarefa, definição versão 1/2, evento legal, URL e GID simulado.<br>
**Caminhos de erro obrigatórios:** constraint, falha transacional, migração, restore, GID duplicado e campo externo conflitado.<br>
**Evidência exigida:** dicionário físico, resultados migration/integration e prova de ausência de conexão.

## Handoff e operação

- **Como demonstrar:** cadastrar projeto/tarefa/definição/evento e reabrir a sessão; conferir ID e histórico.
- **Como operar depois:** aplicação executa migrations versionadas; Admin não gerencia credenciais de conector na Fase 1.
- **Como monitorar:** falhas de gravação/migração e backups verificados.
- **Pendência conhecida:** engine/hosting dependem da stack real da aplicação; integração fica para Fase 2.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| F1-03 | Fechar esquema lógico e contrato de persistência | Engenharia de produto | F1.2 | Entidades têm PK, tipo, FK, índices e constraints definidos | Banco de dados lógico; Integridade | Diagrama e dicionário relacional | Stack/engine identificadas | Planejada |
| F1-09 | Implementar campos externos e connector inativo | Engenharia | F1.2 | GID opcional e mapeamento persistido; sem token ou rede | Contrato preparado; RN-F1.2-02/03 | Schema + inspeção de rede zero | F1-03,F1-15 | Planejada |
| F1-15 | Implementar migrations e persistência do CRUD manual | Engenharia | F1.2 | CRUD relido após sessão; unicidade/FKs válidas | Operação do banco; TDD GREEN | Relatório migration/integration | F1-03 e engine confirmada | Planejada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
