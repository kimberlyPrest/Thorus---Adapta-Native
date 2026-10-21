# Estado atual — Adapta Cliente

- task_id: F1-06
- champion: Amanda (Thórus Admin)
- spec: 04_fase-atual/specs/spec-06-carteira-projetos.md
- etapa: implementando
- autorizacao_implementacao: confirmada + 2026-10-21T11:35 (canal web) — formulário: "Autorizo implementar o plano"
- teste_humano: pendente
- verificacao_automatica: pendente — RED equivalente registrado: /projetos hoje mostra só estado vazio, sem busca/filtros/paginação/autorização; implementação iniciada conforme plano autorizado (fixtures C5, consulta local com semântica C4, URL como fonte de filtros, estados D3)
- aprendizado: pendente
- ultima_acao: autorização recebida; etapa → implementando; alterações restritas à rota /projetos e suporte (lib/fixtures, lib/portfolio, context/DemoRoleContext, shell para compartilhar o perfil de demonstração)
- proxima_acao: implementar carteira, rodar QA pipeline do Skip e parar no teste humano
- atualizado_em: 2026-10-21T11:36:00-03:00