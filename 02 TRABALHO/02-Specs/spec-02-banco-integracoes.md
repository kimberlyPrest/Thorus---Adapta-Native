# SPEC F1.2 — Banco de dados e preparação das integrações

**Fase:** 1 · **Resultado:** aplicação persiste os fluxos manuais e tem contrato de dados pronto para conectar Asana/Drive na Fase 2.  
**Limite:** não autenticar nem fazer chamadas a serviços externos nesta fase.

## Banco e integridade

Implementar o modelo lógico definido em [visão geral Fase 1](spec-00-visao-geral-fase-1.md), incluindo usuários/papéis, clientes, projetos/membros, fases e status, tarefas, definições/versionamento, eventos legais, referências documentais, auditoria, vínculos externos e mapeamentos. Aplicar migrações versionadas e reversíveis sempre que possível, FKs, constraints, índices por filtros usados na carteira e política de arquivamento.

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

## Aceite verificável

- Migrações constroem o banco vazio com todas as FKs, constraints e índices previstos.
- Criar/editar projeto, tarefa, definição, evento legal, referência de documento e usuário grava no banco e continua visível após novo login.
- Um `external_gid` nulo é permitido; repetição de GID no mesmo sistema/workspace/recurso é rejeitada com mensagem compreensível.
- Conflito de campo entre dado manual e valor externo simulado é apresentado como pendência de reconciliação; não sobrescreve o manual.
- Nenhuma requisição DNS/HTTP para Asana/Drive é gerada pelas telas e fluxos da Fase 1.
- Backup restaurado mantém relacionamentos, autoria e histórico conforme política aprovada.

## Perguntas de validação do cliente

Consulte P1 e P2 no [questionário de levantamento](../05-Levantamento/06-Questionario-levantamento-cliente.md). O passo a passo de localizar workspace, projetos, campos, tarefas, status e obter autorização Asana prepara a integração da Fase 2; token não deve ser enviado por formulário ou mensagem.
