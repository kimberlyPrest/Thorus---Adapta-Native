# Estado atual — Adapta Cliente

- task_id: F1-16
- champion: Amanda (Thórus Admin)
- spec: 04_fase-atual/specs/spec-01-layout-navegacao.md
- etapa: aguardando_teste_humano
- autorizacao_implementacao: confirmada 2026-10-29T09:40 (canal web) — formulário: "Autorizo implementar o plano" com escopo "Todas as páginas entregues"
- teste_humano: pendente — roteiro enviado em 2026-10-29 (canal web)
- verificacao_automatica: passou — QA verde v0.0.29 4bf7a5f (setup/static/build/integrações; test ran:false — runner ausente, dívida mantida). Tokens D2 criados em `src/main.css` (`--primary` 175 77% 26% = #0F766E, `--primary-hover` #115E59, `--ring` #0F766E, `--border` #CBD5E1, `--muted-foreground` #475569, semânticos success/warning/info/destructive) e mapeados em `tailwind.config.ts`; tipografia trocada para stack system-ui (import do Google Fonts removido). Hex literais eliminados: 266 ocorrências em 17 cores → 0 nos 7 arquivos (AppShell, Dashboard, Projects, ProjectDetail, ProjectActivity, Profiles, Integrations). Skip-link "Pular para o conteúdo" e `main#conteudo-principal tabIndex=-1` adicionados ao shell. Prova visual no preview: item ativo da sidebar e botões primários em teal, superfícies e textos pelos tokens. Limitação registrada: o engine headless não reporta `getComputedStyle` (retornou vazio), então a prova de cor computada é o screenshot, não o eval; viewports 375/768 e foco por teclado ficam para o teste humano.
- aprendizado: pendente
- ultima_acao: F1-16 implementada e verificada por QA + prova visual; aguardando teste humano
- proxima_acao: aguardar confirmação do teste humano (não concluir nem iniciar outra task antes)
- atualizado_em: 2026-10-29T09:55:00-03:00