# Estado atual — Adapta Cliente

- task_id: F1-02
- champion: Amanda (Thórus Admin)
- spec: 04_fase-atual/specs/spec-04-admin-permissoes.md; P9 afeta também 04_fase-atual/specs/spec-11-alteracoes-decisoes.md
- etapa: liberada para validação pela Thórus
- autorizacao_implementacao: B1 registrado em 2026-10-16 por Produto/Engenharia (06_notas/2026-10-16-registro-b1.md): runtime React+TS (Vite) padrão Skip Cloud + PocketBase, SQLite, migrations em pocketbase/migrations/, test runner Vitest, binding = API de collections + hooks server-side preservando envelope/erros do C3, limite de provisionamento de role conforme C4 (Admin-only, um perfil por usuário, último Admin protegido, access_state pending/ready, policy_version, activity_event). F1-02 valida política; F1-19 implementa a matriz após F1-02 concluída e runtime provisionado.
- teste_humano: nao_aplicavel — nenhuma implementação técnica ocorreu
- verificacao_automatica: pendente — nenhum teste de implementação executado; suítes a mapear por cenário no runner Vitest antes da implementação (C5)
- aprendizado: pendente
- ultima_acao: Em 2026-10-16 Thórus Admin revisou a regra P9: aprovador único designado por projeto/disciplina (Engenharia ou CS), CS apenas avisado; coaprovação obrigatória retirada; DÚVIDA com C2/SPEC-1-011 resolvida sem emenda, pois o contrato já modela decided_by único e criação atômica da versão vigente. P8 integralmente respondido (Admin=all; CS/Engenharia/Legais=assigned; Liderança=all/read-only; documentos liberados consultáveis pelos perfis internos; sem portal; categorias ART/plantas/modelos/memoriais/documentos de aprovação; sem aprovação individual; CS valida contato). Em 2026-10-16 Produto/Engenharia (Adapta) registrou o B1, liberando a F1-02 para validação pela Thórus.
- proxima_acao: Thórus valida e assina a matriz papel × ação × escopo (SPEC-1-004-CA-01); em paralelo, Thórus Admin + Engenharia definem o B3 (método de autenticação corporativa) para destravar F1-04/F1-14
- atualizado_em: 2026-10-16T13:20:00-03:00
