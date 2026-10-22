# Estado atual — Adapta Cliente

- task_id: F1-08
- champion: Amanda (Thórus Admin)
- spec: 04_fase-atual/specs/spec-14-atividade-auditoria.md
- etapa: aguardando_teste_humano
- autorizacao_implementacao: confirmada + 2026-10-22T15:13 (canal web) — formulário: "Autorizo implementar o plano"
- teste_humano: pendente — roteiro enviado em 2026-10-22T15:30 (canal web)
- verificacao_automatica: passou — QA verde v0.0.21 2c1a59a → v0.0.25 06290ca (test ran:false — runner ausente, dívida mantida); migration 0007 aplicada (índice project_id+created em activity_events, coleção idempotency_keys, fixture C5 do evento admin_profiles); rotas provadas por API: lista Admin (2 eventos, filtros resource/actor/data funcionando, total filtrado), CA-03 (CS sem read de admin_profiles não vê o evento — total 1, só projects; detalhe do evento admin → 404 genérico), detalhe seguro (changedFields, sem payload integral), Idempotency-Key no PUT team (1ª execução version 8, replay flag true devolvendo a mesma versão, sem duplicata); evento do PUT team agora grava project_id (escopo da aba); fluxo de UI exercitado: timeline com 2 eventos agrupados por dia, filtro ?resource=projects na URL (1 evento), drawer de detalhe seguro aberto
- aprendizado: pendente
- ultima_acao: F1-08 implementada e verificada (3 bugs corrigidos em revisão: findFirstRecordByFilter lança "no rows" sem try/catch; evento do PUT team sem project_id; import duplicado de Link); aguardando teste humano
- proxima_acao: aguardar confirmação do teste humano (não concluir nem iniciar outra task antes)
- atualizado_em: 2026-10-22T15:30:00-03:00