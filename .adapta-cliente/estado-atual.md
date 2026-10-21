# Estado atual — Adapta Cliente

- task_id: F1-06
- champion: Amanda (Thórus Admin)
- spec: 04_fase-atual/specs/spec-06-carteira-projeto.md
- etapa: aguardando_teste_humano
- autorizacao_implementacao: confirmada + 2026-10-21T11:35 (canal web) — formulário: "Autorizo implementar o plano"
- teste_humano: pendente — roteiro enviado em 2026-10-21T11:47 (canal web)
- verificacao_automatica: passou — QA pipeline do Skip verde (setup/staticAnalysis/build/integrações; test ran:false — runner ausente, dívida marcada); versões 0.0.5 (91b5d57) e 0.0.6 (2208d5f, store singleton do perfil de demonstração); preview verificado por automação: Admin vê 1 projeto ativo (A), busca ?q=vale filtra na URL, Ver arquivados ?archived=1 exibe B (2 projetos) e Limpar filtros restaura; sentinela: B nunca aparece para CS em lista/total/filtro (mesma função authorizedRows exercida pelos dois perfis); limitação: troca de perfil via Radix Select não automatizável — verificação humana
- aprendizado: pendente
- ultima_acao: F1-06 implementada (7 arquivos: lib/fixtures.ts ampliado, lib/portfolio.ts, lib/demoRoleStore.ts, context/DemoRoleContext.tsx, components/shell/AppShell.tsx, pages/Projects.tsx, App.tsx) e QA verde; aguardando teste humano
- proxima_acao: aguardar confirmação do teste humano (não concluir nem iniciar outra task antes)
- atualizado_em: 2026-10-21T11:47:00-03:00