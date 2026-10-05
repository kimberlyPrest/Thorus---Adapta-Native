# Contrato de dados e API local — Fase 1

**Revisão:** 05/10/2026. Este documento fixa o contrato lógico comum. Endpoints HTTP abaixo são contratos da aplicação proposta, não APIs já existentes. Se o framework usar ações servidor em vez de REST, preservar entradas, saídas, autorização e erros; registrar o binding concreto antes de implementar.

## C1 — Decisões técnicas e fonte de verdade

O repositório atual contém documentação, sem aplicativo, engine de banco ou provedor de sessão definidos. **B1:** Engenharia de produto escolhe/identifica stack, banco relacional, ferramenta de migração e comandos de teste antes da primeira task que grava código. **B3:** Thórus Admin + Engenharia escolhem o método de autenticação corporativa antes de contas reais. Estes pontos são decisões técnicas explícitas, não campos a adivinhar no desenvolvimento. O desenho de página, contrato lógico e fixtures já estão definidos nesta revisão.

Banco da aplicação é fonte de verdade do MVP manual. IDs são UUID; datas de calendário são DATE; instantes são timestamp UTC. Todos os registros editáveis têm `version` inteiro (inicial 1), `created_at`, `updated_at` e `created_by`. Arquivamento/desativação preserva FK e histórico. Cada SPEC cria sua persistência no mesmo incremento de interface.

## C2 — Entidades, chaves e validações

| Entidade | Campos e integridade mínima |
|---|---|
| users | id, name 2–120, email normalizado ≤254 UNIQUE, identity_subject opcional UNIQUE, role_id FK, active boolean, access_state pending/ready; identidade nunca inferida por nome/cargo |
| roles | id, key UNIQUE entre admin/cs/engineering/legal/leadership, label, policy_version; um perfil por usuário no MVP |
| role_permissions | role_id, resource, action, scope assigned/all; PK(role_id,resource,action); ausência da linha = negar |
| project_memberships | project_id, user_id, active; UNIQUE(project_id,user_id); membro inativo preservado para histórico |
| project_approvers | project_id, discipline_id, user_id, active; autoridade técnica exige usuário ativo/membro e permissão definitions.decide |
| clients | id, name 2–160 obrigatório; normalized_name para busca; cadastro sem CPF/CNPJ/endereço/contato pessoal obrigatório |
| projects | id, code opcional ≤30 UNIQUE se preenchido (normalizar maiúscula/trim), name 2–160, client_id FK, city opcional ≤120, state_code opcional enum UF, contracted_scope opcional ≤4000, phase_option_id/status_option_id FK opcionais, start_date/target_date DATE opcionais, cs_user_id/technical_user_id FK opcionais, notes ≤4000, archived_at opcional; target_date ≥ start_date quando ambas existirem |
| catalog_options | id, kind, key UNIQUE(kind,key), label 1–80, position inteiro ≥0, active; kinds project_phase/project_status/discipline/priority/legal_type/legal_status/document_category; não misturar catálogo com estados executáveis |
| project_phases | id, project_id FK, catalog_option_id FK, position, active; UNIQUE(project_id,catalog_option_id); base por projeto, independente de template futuro |
| tasks | id, project_id, phase_id opcional mesmo projeto, title 2–200, description ≤4000, assignee_id opcional membro ativo, priority_option_id opcional, due_date DATE opcional, state todo/in_progress/done, completed_at opcional; done exige completed_at do servidor |
| technical_definitions | id, project_id, discipline_option_id, key 1–80, label 2–160, current_version_id FK; UNIQUE(project_id,discipline_option_id,key normalizada) |
| definition_versions | id, definition_id, number positivo UNIQUE(definition_id,number), value ≤4000, observation ≤2000, decided_by, decided_at, request_id opcional; imutável; uma vigente via ponteiro current_version_id, não múltiplos flags concorrentes |
| change_requests | id, definition_id, base_version_id FK, proposed_value 1–4000, reason 1–2000, impact_note opcional ≤2000, origin manual/meeting_reference/client_request, source_channel whatsapp/email/meeting/phone/in_person opcional, source_url opcional, state draft/pending/approved/rejected/cancelled, requested_by, decided_by/at/note opcionais, version; proposta sempre ligada à versão-base |
| legal_events | id, project_id, type_option_id, status_option_id, responsible_id membro ativo, event_date DATE, protocol opcional ≤100, note ≤4000, evidence_url opcional ≤2048; confirmações via flag confirmed_by/confirmed_at; atualizar fato confirmado invalida confirmação |
| document_references | id, project_id, name 2–200, category_option_id opcional, url HTTP(S) ≤2048, observation ≤2000, archived_at opcional, origin=manual; acesso interno herdado do projeto; sem binário |
| activity_events | id, project_id opcional, actor_id, resource/action, target_id, occurred_at UTC, result, changed_field_names, safe_summary, correlation_id; append-only; não duplicar textos integrais de definição em log genérico |
| external_links | id, project_id, system asana/drive, workspace_id opcional, resource_type, gid opcional, permalink opcional; UNIQUE(system,coalesce(workspace_id,''),resource_type,gid) para gid não nulo; escopo preparação |
| integration_mappings | id, system, resource_type, external_field_key, internal_field_key, transform identity/text/enum_mapping/date, enum_map JSON se aplicável, revision, review_state draft/reviewed, enabled=false; sem segredo e sem execução |
| client_requests | id, project_id, requester_contact_id/label opcionais, intake_channel whatsapp/email/phone/meeting/in_person, request_type info/document/technical_change/other, summary 1–2000 sem conversa integral, source_reference URL opcional, assignee_id, state new/in_progress/waiting_confirmation/waiting_customer/completed/cancelled, due_date DATE opcional, version e timestamps |
| customer_updates | id, project_id, source_event_id, source_version, recipient_contact_id, channel whatsapp_manual, message_body 1–4000, state draft/review/approved_for_manual_send/manually_sent/cancelled, approved_by/at, manually_sent_by/at, version; uma atualização ativa por evento/contato/versão |
| mutation_receipts | actor_id, action_key, request_id, payload_hash, response_data, created_at; UNIQUE(actor_id,action_key,request_id); retenção de 24h, sem incluir credenciais |

Índices: projects por client/phase/status/archived; memberships por user/project; tasks por project/state/due_date/assignee; changes por state/definition; events por project/event_date; activity por project/occurred_at. FKs de histórico usam RESTRICT, nunca cascade que apague decisões. Campos de texto simples não aceitam HTML executável.

## C3 — Protocolo de consulta e gravação

- Base `/api`. Resposta de sucesso `{success:true,data,meta:{page,pageSize,total,correlationId}}`; resposta de erro `{success:false,error:{code,message,fields},meta:{correlationId}}`. Detalhes técnicos ficam no servidor; fields contém somente erros de campo autorizados.
- Coleções aceitam page≥1, pageSize 25/50, sort com allowlist por endpoint; empates por id. Autorização deve restringir query antes de busca, paginação, aggregates e contagem.
- POST e ações de transição aceitam header `Idempotency-Key` (UUID por clique); PATCH aceita `expectedVersion`. Reenvio de mesma chave/payload retorna mesmo resultado; mesma chave/payload diferente retorna 409. Sem chave duplicada, constraint por entidade continua obrigatória.
- Sucesso POST=201; GET/PATCH/transição=200. 401 sem sessão, 403 função negada em recurso visível, 404 registro/projeto invisível ou inexistente, 409 conflito/duplicidade protegida, 422 campos inválidos, 503 indisponível. E-mail/código duplicado só é explicado ao ator autorizado a administrá-lo.
- Em edição versionada, UPDATE exige versão atual; conflito não altera dados. Conservar rascunho e buscar snapshot atualizado para comparar/revisar. Após revisão, usuário confirma novo submit com versão nova; não fazer retry automático de PATCH.
- Mutação e evento de auditoria ficam na mesma transação; falha na auditoria impede commit. Alteração técnica aprovada inclui versão, ponteiro vigente, decisão da solicitação e evento na mesma transação. Retry de escrita só por confirmação do usuário, reaproveitando chave quando o resultado anterior estiver desconhecido.
- URL: HTTP(S), rejeitar javascript/data/file, credenciais embutidas e caracteres de controle. Sem fetch/preview de servidor, sem provar ACL. Texto livre escapado; rate-limit auth/mutações conforme runtime B1, documentando configuração concreta antes do aceite técnico.

## C4 — Política de autorização

Um usuário tem um perfil ativo. A permissão efetiva exige conta ativa + access_state=ready + sessão válida + recurso/ação no perfil + escopo de projeto. `assigned` exige project_memberships ativa; `all` cobre todos os projetos da organização. ID conhecido não concede leitura. Admin pode gerir cadastros, mas aprovação técnica exige designação explícita em project_approvers.

| Recurso | Ações editáveis na matriz |
|---|---|
| projects | read/create/update/archive/manage_team |
| tasks | read/create/update/complete |
| definitions | read/register_initial/propose/decide |
| legal_events | read/create/update/confirm |
| documents | read/create/update/archive |
| activity | read |
| dashboard | read (métricas só dos recursos com read e do escopo permitido) |
| admin_users | read/create/update/deactivate (Admin fixo) |
| admin_profiles | read/update (Admin fixo) |
| admin_lists | read/update (Admin fixo) |
| admin_mappings | read/update (Admin fixo) |
| client_requests | read/create/update/assign/close (CS/assigned; Admin/all) |
| customer_updates | read/create/update/review/record_manual_send (CS/assigned) |

Permissão de escrever exige read do mesmo recurso e projects.read; se checkbox write for marcado, interface inclui read explicitamente no resumo antes de salvar. Create projeto não exige projeto prévio: se scope assigned, criador vira membro automaticamente. projects.create fica limitado a Admin/CS na base, sem exigir cliente real para demo.

**Base proposta para o piloto (P8 confirma antes de contas reais):** Admin todos os cadastros/administrativos; CS assigned com projects CRUD/equipe, tasks CRUD, definitions read/propose, legal_events read/create/update, documents CRUD e activity/dashboard read; Engenharia assigned projects read, tasks CRUD, definitions read/register_initial/propose (decide só se designado), documents/activity/dashboard read; Legais assigned projects read, tasks CRUD, definitions read, legal_events CRUD/confirm, documents CRUD, activity/dashboard read; Liderança all somente read dos recursos. Nenhum perfil de cliente existe na Fase 1. Ajustes são realizados pela matriz da SPEC-1-004.

Admin mantém acesso de administração reservado; protegê-lo contra remover toda capacidade de gestão. Desativação do último Admin ativo ou auto-desativação é bloqueada. Mudança de política incrementa policy_version; servidor consulta versão atual em toda requisição para que a próxima operação aplique a regra nova. Frontend atualiza capacidades após salvar/ao receber 403; dados já vistos não podem ser “desvistos”, mas novos endpoints são bloqueados imediatamente.

## C5 — Fixtures e evidências

Projetos fictícios A e B, Admin, CS atribuído só a A, Engenharia em A, Legais em A, Liderança, usuário desativado. Em A: tarefa vencida ontem, tarefa hoje, tarefa em +7 dias, tarefa +8 dias, concluída e sem prazo; definição Elétrica/Carga instalada v1=100 kVA; proposta v2=120 kVA; evento legal e URL de exemplo. Em B: sentinela para comprovar que query/contagem não vazam metadados.

Antes da implementação, mapear suítes por cenário para o runner decidido em B1. SPECs definem ações verificáveis de teste; não inventar comandos npm/pnpm sem projeto. Para cada aceite, registrar cenário, evidência UI, resposta API redigida e releitura do banco. Backup/restore é em ambiente isolado, usando fixtures, nunca restauração sobre produção por este plano.
