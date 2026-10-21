# Status do projeto

**Data:** 21/10/2026<br>
**Estado:** F1-01, F1-02, F1-03, F1-05, F1-06 e F1-09 concluídas (6/41, ~15%) — dicionário de campos aprovado (SPEC-1-007-CA-01), matriz de acesso validada (SPEC-1-004-CA-01), modelo relacional aprovado (SPEC-1-007-CA-02), shell (SPEC-1-001), carteira autorizada (SPEC-1-006-CA-01) e mapeamentos de integração preparados sem egress (SPEC-1-016-CA-01).

Na Fase 1, a equipe registra e atende pedidos de clientes, envia documentos autorizados manualmente, prepara/revisa e registra atualizações de eventos confirmados e centraliza alterações técnicas recebidas por diferentes canais. As integrações Asana, Drive, sistema Legais e WhatsApp ficam para Fase 2. A credencial local do Asana foi usada somente em consultas GET de leitura. Nenhum dado foi escrito no Asana ou salvo como exportação no pacote. Remova a credencial do `.env` e revogue-a após esta revisão.

**Decisões técnicas (B1, 16/10):** runtime React+TS (Vite) padrão Skip Cloud + PocketBase; SQLite; migrations em `pocketbase/migrations/`; test runner Vitest; binding = API de collections + hooks server-side preservando o contrato C3; limite de provisionamento de role conforme C4. Detalhes em `06_notas/2026-10-16-registro-b1.md`.

**Matriz de acesso (F1-02, 20/10):** Admin=all; CS, Engenharia e Legais=assigned; Liderança=all somente leitura; recursos/ações conforme C4; escrita exige read + projects.read; um perfil por usuário; aprovação técnica exige designação por projeto/disciplina (regra P9 revisada em 16/10).

**Dicionário de campos (F1-01, 21/10):** mínimo obrigatório nome + cliente; demais campos do C2 opcionais; nenhum dado pessoal obrigatório. Registro em `06_notas/2026-10-21-registro-f1-01-dicionario.md`. P1/P2 "ainda não definido" — bloqueiam apenas a integração da Fase 2.

**Modelo relacional (F1-03, 21/10):** clients/projects com versionamento C2 (version inicial 1), FKs RESTRICT, unicidade de code (quando preenchido) e normalized_name, índices de consulta aprovados; provisionamento por migration versionada (padrão B1). Registro em `06_notas/2026-10-21-registro-f1-03-modelo-relacional.md`. DÚVIDA aberta: unicidade de normalized_name diverge da permissão de nomes iguais em SPEC-1-007 — o consultor decide por emenda antes da migration (F1-15).

**Shell (F1-05, 21/10):** primeira implementação no app Skip (v0.0.4 `8ac2ed5`) — sidebar com menu por perfil, submenu Configurações, rotas futuras desabilitadas ("Em breve"), breadcrumb, drawer mobile, Dashboard de referência D2/D3; identidade de demonstração por fixture até /api/me (SPEC-1-002). Teste humano aprovado pelo champion.

**Carteira (F1-06, 21/10):** rota `/projetos` (v0.0.7 `7f71374`) — busca normalizada com debounce, filtros de cliente/fase/situação/CS/responsável/UF, arquivados, ordenação e paginação 25/50, tudo na URL; autorização local com semântica C4 sobre fixtures C5 (projeto B sentinela nunca vaza para CS). SPEC-1-006-CA-01 fechada; teste humano aprovado.

**Integrações planejadas (F1-09, 21/10):** primeira entrega com banco real no app Skip (v0.0.12 `5e3355a`) — migrations 0001–0003 criam `integration_mappings` e `integration_links` (regras Admin-only C4, versionamento C2) e o campo `role` em `users`; hooks server-side forçam `enabled=false`, recusam campo interno fora da allowlist, GID duplicado e revision obsoleta; página `/configuracoes/integracoes` com banner "Fase 2", sem botão Conectar/Sincronizar/Testar/Upload e sem campo de token. Provas de API + teste humano aprovado pelo champion. Dívidas: project_id texto até a F1-15; role admin/cs até a F1-19; auditoria na F1-08.

**Próximas dependências:** F1-33 (mapeamento draft/review com rejeições) liberada — pré-condições F1-03/F1-09/F1-15 parcialmente satisfeitas (F1-15 segue pendente da DÚVIDA do normalized_name — conferir com o consultor antes de selecionar); F1-15 depende da DÚVIDA do normalized_name; F1-19 planejável após runtime provisionado; F1-04/F1-14 aguardam B3.

- [Escopo](01_projeto/escopo-definitivo.md)
- [Tasks Fase 1](04_fase-atual/fase.md)
- [SPECs](04_fase-atual/specs/00-INDICE.md)
- [Rastreabilidade](04_fase-atual/matriz-de-rastreabilidade.md)
- [Questionário](03_documentos/06-Questionario-levantamento-cliente.md)
- [Registro B1](06_notas/2026-10-16-registro-b1.md)
