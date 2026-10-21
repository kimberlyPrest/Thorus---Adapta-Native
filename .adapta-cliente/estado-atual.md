# Estado atual — Adapta Cliente

- task_id: F1-19
- champion: Amanda (Thórus Admin)
- spec: 04_fase-atual/specs/spec-04-admin-permissoes.md
- etapa: aguardando_teste_humano
- autorizacao_implementacao: confirmada + 2026-10-21T13:57 (canal web) — formulário: "Autorizo implementar o plano"
- teste_humano: pendente — roteiro enviado em 2026-10-21T14:15 (canal web)
- verificacao_automatica: passou — QA pipeline verde (v0.0.13 e8d372e, v0.0.14 f0edc9b; test ran:false — runner ausente, dívida mantida); migrations 0004 (roles/role_permissions/policy_state/activity_events) e 0005 (seed da matriz assinada: admin 36, cs 13, engenharia 10, legais 6, liderança 7 grants) aplicadas; rotas provadas por API: preview==delta salvo, salvar incrementa policy_version (1→2), reenvio obsoleto recusado, write sem read recusado, motivo obrigatório, CS/anônimo 403; evento de auditoria gravado em activity_events na mesma transação; matriz do CS restaurada (policy 4, tasks:complete de volta); página /configuracoes/perfis verificada autenticada (matriz, abas, escopos) e com gate pelo auth real (não-Admin vê "Ação indisponível")
- aprendizado: pendente
- ultima_acao: F1-19 implementada e verificada; aguardando teste humano
- proxima_acao: aguardar confirmação do teste humano (não concluir nem iniciar outra task antes)
- atualizado_em: 2026-10-21T14:15:00-03:00