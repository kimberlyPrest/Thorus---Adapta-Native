# Estado atual — Adapta Cliente

- task_id: F1-09
- champion: Amanda (Thórus Admin)
- spec: 04_fase-atual/specs/spec-16-admin-integracoes-planejadas.md
- etapa: aguardando_teste_humano
- autorizacao_implementacao: confirmada + 2026-10-21T12:11 (canal web) — formulário: "Autorizo implementar o plano"
- teste_humano: pendente — roteiro enviado em 2026-10-21T12:30 (canal web)
- verificacao_automatica: passou — QA pipeline verde (v0.0.9 4a80144, v0.0.10 4ac4cc5, v0.0.11 29febe6, v0.0.12 5e3355a; test ran:false — runner ausente, dívida mantida); migrations 0001/0002/0003 aplicadas (integration_mappings, integration_links, campo role em users); API exercitada: mapeamento criado com enabled=false forçado pelo hook, enabled=true recusado, campo interno fora da allowlist recusado, sem token recusado, revision C2 (update 2 ok, obsoleta recusada), GID duplicado recusado; preview verificado autenticado como Admin (banner Fase 2, tabelas com dados, sem botão Conectar/Sincronizar/Testar/Upload, sem input de token) e não autenticado (listas vazias, sem vazamento)
- aprendizado: pendente
- ultima_acao: F1-09 implementada (migration 0001 coleções + 0002 role em users + 0003 correção BoolField required; hooks model com validações inline; service integrations.ts; página Integrations.tsx; rota e menu habilitados) e verificada; aguardando teste humano
- proxima_acao: aguardar confirmação do teste humano (não concluir nem iniciar outra task antes)
- atualizado_em: 2026-10-21T12:30:00-03:00