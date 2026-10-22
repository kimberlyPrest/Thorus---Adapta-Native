# Estado atual — Adapta Cliente

- task_id: F1-08
- champion: Amanda (Thórus Admin)
- spec: 04_fase-atual/specs/spec-14-atividade-auditoria.md
- etapa: concluida
- autorizacao_implementacao: confirmada 2026-10-22T15:13 (canal web) — formulário: "Autorizo implementar o plano"
- teste_humano: aprovado 2026-10-22T15:34 (canal web) — formulário: "Funcionou"
- verificacao_final: passou — 5 provas de API reprovadas do zero (logins novos): lista Admin 200; CS membro vê só projects (CA-03); filtro admin_profiles do CS não vaza; anônimo 401; PUT versão obsoleta + chave nova → 409 sem consumir chave. QA verde v0.0.21 2c1a59a → v0.0.25 06290ca.
- fechamento: fase.md F1-08 → Concluída (2026-10-22) — commit 3ee73de6 (21005 bytes, 41 tasks, validado byte a byte; incidente 4 de truncamento corrigido — ver controle.md); changelog 74f6884d; STATUS 2e50bed7; controle.md (aprendizados JSVM + incidente 4).
- aprendizado: registrado (2 linhas em 06_notas/aprendizado-continuo/controle.md)
- ultima_acao: fechamento documental completo da F1-08
- proxima_acao: sem_task — próximas candidatas: F1-30 (elegível, depende F1-08 ✓ + F1-22 ✓) e F1-33 (conferir F1-15/DÚVIDA do normalized_name com o consultor antes de selecionar)
- atualizado_em: 2026-10-22T15:58:00-03:00