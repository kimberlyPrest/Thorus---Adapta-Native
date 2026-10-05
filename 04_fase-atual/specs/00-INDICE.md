# Índice de SPECs — Fase 1 vertical

**Resultado da fase:** MVP interno manual com banco relacional e interface utilizáveis para operar projeto, equipe, tarefa, alteração técnica, evento legal e referência documental.

As SPECs a seguir são fatias verticais: cada uma entrega tela/rota, dados, validação no servidor, grants, estados, auditoria e prova da própria ação. As decisões visuais e de dados compartilhadas estão nos dois contratos comuns. Cada página só é aceita quando a interface, a persistência e a autorização no servidor funcionam em conjunto.

## Contratos comuns

| Referência | Função |
|---|---|
| [Contrato de interface D1–D5](01-contrato-interface.md) | Sidebar, rotas, tokens, componentes, estados, responsividade e acessibilidade |
| [Contrato de dados/API C1–C5](02-contrato-dados-api.md) | Banco lógico, envelope de API, transação, conflito, idempotência, permissões e fixtures |

## Rotas e fluxos entregues

| SPEC | Rota/entrega | Resultado observável |
|---|---|---|
| [SPEC-1-001 — Layout, sidebar e navegação aplicada](spec-01-layout-navegacao.md) | `/dashboard e /projetos (shell)` | Navegar entre Dashboard/Projetos e aba do projeto entregue mantendo URL e foco. |
| [SPEC-1-002 — Entrada, sessão e saída do usuário](spec-02-entrada-sessao.md) | `/entrar` | Sair revoga sessão; tentar GET anterior exige autenticação. |
| [SPEC-1-003 — Administração de usuários pela interface](spec-03-admin-usuarios.md) | `/configuracoes/usuarios` | Admin edita vínculo, desativa usuário de teste e verifica que sua próxima ação é recusada. |
| [SPEC-1-004 — Administração de funções e níveis de permissão](spec-04-admin-permissoes.md) | `/configuracoes/perfis` | CS tenta concluir tarefa pela UI e por chamada direta; ambas negam na requisição seguinte. |
| [SPEC-1-005 — Listas operacionais editáveis pelo Admin](spec-05-admin-listas.md) | `/configuracoes/listas` | Desativar opção a remove de novas seleções; registro antigo continua legível. |
| [SPEC-1-006 — Carteira: busca, filtros e paginação](spec-06-carteira-projetos.md) | `/projetos` | Limpa filtros, usa arquivados e compara com Liderança autorizada a todos. |
| [SPEC-1-007 — Novo e editar projeto com cliente e salvamento](spec-07-cadastro-projeto.md) | `/projetos/novo e /projetos/{id}/editar` | Editar muda escopo/data; concorrência compara versão e mantém rascunho. |
| [SPEC-1-008 — Visão geral, equipe e arquivamento do projeto](spec-08-projeto-visao-equipe.md) | `/projetos/{id}` | Arquivar confirma suspensão de gravação; Reativar recupera operação sem excluir dados. |
| [SPEC-1-009 — Aba Tarefas e fases: registrar e acompanhar trabalho](spec-09-tarefas-fases.md) | `/projetos/{id}/tarefas` | Filtrar vencidas, próximos prazos e sem prazo usando fixtures C5. |
| [SPEC-1-010 — Aba Definições: vigente e histórico por disciplina](spec-10-definicoes-vigentes.md) | `/projetos/{id}/definicoes` | Seleciona Propor alteração e inicia pedido separado com baseVersion atual. |
| [SPEC-1-011 — Solicitar, revisar e decidir alteração técnica](spec-11-alteracoes-decisoes.md) | `/projetos/{id}/definicoes/solicitacoes` | Após aprovar, voltar às Vigentes mostra v2 e histórico v1; repetir decisão/reenvio não duplica versão. |
| [SPEC-1-012 — Aba Legais e marcos com evidências e confirmação](spec-12-legais-marcos.md) | `/projetos/{id}/legais` | Editar dado confirmado exige nova confirmação e mostra histórico de correção. |
| [SPEC-1-013 — Aba Documentos: referências manuais e organização](spec-13-documentos-referencias.md) | `/projetos/{id}/documentos` | Editar nome ou arquivar sem afetar documento de origem. |
| [SPEC-1-014 — Aba Atividade: linha do tempo e detalhes da mudança](spec-14-atividade-auditoria.md) | `/projetos/{id}/atividade` | Negar read do recurso e comprovar retirada do evento e total na próxima consulta. |
| [SPEC-1-015 — Dashboard acionável com métricas e detalhes](spec-15-dashboard.md) | `/dashboard e /dashboard/{tarefas|alteracoes}` | Mudar tarefa, atualizar e comparar com usuário que não pode ler tarefas. |
| [SPEC-1-016 — Administração de mapeamentos preparados para Fase 2](spec-16-admin-integracoes-planejadas.md) | `/configuracoes/integracoes` | Recarrega tela e comprova enabled=false e ausência de chamada ao serviço externo. |
| [SPEC-1-017 — Piloto manual, persistência e recuperação comprovados](spec-17-piloto-recuperacao.md) | `Roteiro das rotas entregues + ambiente isolado de backup` | Backup/restaurar no ambiente isolado e comparar relações/versões/contagens; colher aceite humano da fase. |

## Ordem de dependência

1. D1–D5 e C1–C5 são decisões comuns. Engenharia identifica runtime/banco/test runner B1 antes de executar a primeira task de código; Thórus confirma B3/P8 e as decisões P5/P9 antes de operar dados reais.
2. Shell/sessão/Admin (001–005); banco/cadastro/carteira/equipe (006–008); tarefas, definições, pedidos, legais, documentos e atividade (009–014); dashboard e preparo externo desligado (015–016).
3. Piloto e backup isolado (017) depois das rotas manuais.

**Limites:** Asana/Drive/Gemini/WhatsApp de negócio não recebem chamadas nem sincronizam na Fase 1. Endpoints `/api` descritos são contrato local da aplicação; a implementação confirma o binding específico após B1, sem mudar campos, autorização, erro e critérios sem emenda.
