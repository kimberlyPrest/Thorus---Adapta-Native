# Status do projeto

**Data:** 21/10/2026<br>
**Estado:** F1-01, F1-02, F1-03, F1-05 e F1-06 concluídas (5/41, ~12%) — dicionário de campos aprovado (SPEC-1-007-CA-01), matriz de acesso validada e assinada (SPEC-1-004-CA-01), modelo relacional aprovado para migração (SPEC-1-007-CA-02), shell com sidebar/breadcrumb (SPEC-1-001) e carteira com busca, filtros e paginação autorizados (SPEC-1-006-CA-01) implementados e testados no preview.

Na Fase 1, a equipe registra e atende pedidos de clientes, envia documentos autorizados manualmente, prepara/revisa e registra atualizações de eventos confirmados e centraliza alterações técnicas recebidas por diferentes canais. As integrações Asana, Drive, sistema Legais e WhatsApp ficam para Fase 2. A credencial local do Asana foi usada somente em consultas GET de leitura. Nenhum dado foi escrito no Asana ou salvo como exportação no pacote. Remova a credencial do `.env` e revogue-a após esta revisão.

**Decisões técnicas (B1, 16/10):** runtime React+TS (Vite) padrão Skip Cloud + PocketBase; SQLite; migrations em `pocketbase/migrations/`; test runner Vitest; binding = API de collections + hooks server-side preservando o contrato C3; limite de provisionamento de role conforme C4. Detalhes em `06_notas/2026-10-16-registro-b1.md`.

**Matriz de acesso (F1-02, 20/10):** Admin=all; CS, Engenharia e Legais=assigned; Liderança=all somente leitura; recursos/ações conforme C4; escrita exige read + projects.read; um perfil por usuário; aprovação técnica exige designação por projeto/disciplina (regra P9 revisada em 16/10).

**Dicionário de campos (F1-01, 21/10):** mínimo obrigatório nome + cliente; demais campos do C2 opcionais; nenhum dado pessoal obrigatório. Registro em `06_notas/2026-10-21-registro-f1-01-dicionario.md`. P1/P2 "ainda não definido" — bloqueiam apenas a integração da Fase 2.

**Modelo relacional (F1-03, 21/10):** clients/projects com versionamento C2 (version inicial 1), FKs RESTRICT, unicidade de code (quando preenchido) e normalized_name, índices de consulta aprovados; provisionamento por migration versionada (padrão B1). Registro em `06_notas/2026-10-21-registro-f1-03-modelo-relacional.md`. DÚVIDA aberta: unicidade de normalized_name diverge da permissão de nomes iguais em SPEC-1-007 — o consultor decide por emenda antes da migration (F1-15).

**Shell (F1-05, 21/10):** primeira implementação no app Skip (v0.0.4 `8ac2ed5`) — sidebar com menu por perfil (Admin/CS), submenu Configurações, rotas futuras desabilitadas ("Em breve"), breadcrumb, drawer mobile, página de referência D2/D3 no Dashboard e Projetos com estado vazio; identidade de demonstração por fixture (a SPEC-1-001 permite até /api/me existir); QA da plataforma verde; suite de testes pendente de runner (dívida marcada). Teste humano aprovado pelo champion.

**Carteira (F1-06, 21/10):** rota `/projetos` no app Skip (v0.0.7 `7f71374`) — busca normalizada (debounce 300 ms, Enter imediato), filtros de cliente, fase, situação, CS, responsável técnico e UF (opções derivadas só do universo autorizado), ver arquivados, limpar filtros, ordenação nome/atualizado/próximo prazo e paginação 25/50 — tudo persistido na URL. Autorização local com semântica C4 sobre fixtures C5 (projetos A e B; B é sentinela — nunca aparece para o CS em lista, total ou filtro). Teste humano aprovado pelo champion; SPEC-1-006-CA-01 fechada. Dívidas: dados e autorização locais até a migration (F1-15) e o binding C3 real; ações por linha (Abrir/Editar/Arquivar) para a SPEC-1-008.

**Próximas dependências:** F1-22 (validação de filtros, privacidade e estados de retorno da carteira) está liberada — pré-condição F1-06 satisfeita; F1-15 depende da DÚVIDA do normalized_name; F1-19 planejável após runtime provisionado; F1-04/F1-14 aguardam B3 (autenticação corporativa — decisão Thórus + Engenharia).

- [Escopo](01_projeto/escopo-definitivo.md)
- [Tasks Fase 1](04_fase-atual/fase.md)
- [SPECs](04_fase-atual/specs/00-INDICE.md)
- [Rastreabilidade](04_fase-atual/matriz-de-rastreabilidade.md)
- [Questionário](03_documentos/06-Questionario-levantamento-cliente.md)
- [Registro B1](06_notas/2026-10-16-registro-b1.md)
