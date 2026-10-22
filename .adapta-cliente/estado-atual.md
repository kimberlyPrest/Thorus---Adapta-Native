# Estado atual — Adapta Cliente

- task_id: F1-24
- champion: Amanda (Thórus Admin)
- spec: 04_fase-atual/specs/spec-08-projeto-visao-equipe.md
- etapa: aguardando_teste_humano
- autorizacao_implementacao: confirmada + 2026-10-22T11:52 (canal web) — formulário: "Autorizo implementar o plano"
- teste_humano: pendente — roteiro enviado em 2026-10-22T12:05 (canal web)
- verificacao_automatica: passou — QA verde v0.0.16 3b80f6f → v0.0.20 05f156a (test ran:false — runner ausente, dívida mantida); migration 0006 aplicada (project_memberships, project_approvers, project_state + seed); rotas provadas por API: GET detalhe (Admin 200, CS membro 200, CS sem vínculo 404 genérico, anônimo 401), PUT team transacional (membro +1, version 1→2, 409 em versão obsoleta, 403 não-Admin, aprovador sem definitions.decide recusado, membro removido perde acesso na próxima requisição, evento de auditoria gravado); fluxo de UI exercitado no preview: drawer aberto, aprovador designado por disciplina, salvar → "Equipe salva — versão 5", badge "Aprovador · estruturas" visível, persistência confirmada por releitura de API
- aprendizado: pendente
- ultima_acao: F1-24 implementada e verificada (2 bugs corrigidos em revisão: spread de objeto não suportado no JSVM; ordem limit/offset em findRecordsByFilter); aguardando teste humano
- proxima_acao: aguardar confirmação do teste humano (não concluir nem iniciar outra task antes)
- atualizado_em: 2026-10-22T12:05:00-03:00