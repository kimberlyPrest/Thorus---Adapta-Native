# Estado atual — Adapta Cliente

- task_id: F1-19
- champion: Amanda (Thórus Admin)
- spec: 04_fase-atual/specs/spec-04-admin-permissoes.md
- etapa: implementando
- autorizacao_implementacao: confirmada + 2026-10-21T13:57 (canal web) — formulário: "Autorizo implementar o plano"
- teste_humano: pendente
- verificacao_automatica: pendente — RED equivalente registrado: não existem coleções roles/role_permissions/policy_state/activity_events nem rotas de edição de matriz; /configuracoes/perfis é item "Em breve"; implementação iniciada conforme plano autorizado (migration 0004 schema + 0005 seed da matriz assinada, 4 rotas /backend/v1/admin/*, página Admin-only)
- aprendizado: pendente
- ultima_acao: autorização recebida; etapa → implementando; alterações restritas à rota /configuracoes/perfis e suporte (migrations, hooks de rota, service, página, menu)
- proxima_acao: implementar, rodar QA pipeline do Skip e parar no teste humano
- atualizado_em: 2026-10-21T13:58:00-03:00