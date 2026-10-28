# Estado atual — Adapta Cliente

- task_id: F1-16
- champion: Amanda (Thórus Admin)
- spec: 04_fase-atual/specs/spec-01-layout-navegacao.md
- etapa: aguardando_autorizacao
- autorizacao_implementacao: ausente
- teste_humano: pendente
- verificacao_automatica: pendente — baseline RED provado: tokens D2 inexistentes. `src/main.css` mantém `--primary: 221.2 83.2% 53.3%` (azul #2563EB do template shadcn), enquanto o contrato D2 exige ação primária #0F766E. As páginas aplicam D2 por classe arbitrária (AppShell: 39 ocorrências de hex, 6 cores distintas: #0F766E 14x, #0F172A 9x, #475569 8x, #CBD5E1 6x, #F8FAFC, #94A3B8) — trocar o tema exige editar cada arquivo. Tipografia: contrato pede stack system-ui, config põe Inter primeiro.
- aprendizado: pendente
- ultima_acao: F1-16 selecionada e analisada (proxima-task); relatório com formulário de decisão enviado no canal web
- proxima_acao: aguardar autorização para implementar
- atualizado_em: 2026-10-28T15:58:00-03:00