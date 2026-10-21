# Status do projeto

**Data:** 21/10/2026<br>
**Estado:** F1-01, F1-02 e F1-03 concluídas — dicionário de campos aprovado (SPEC-1-007-CA-01), matriz de acesso validada e assinada (SPEC-1-004-CA-01) e modelo relacional de clients/projects aprovado para migração (SPEC-1-007-CA-02).

Na Fase 1, a equipe registra e atende pedidos de clientes, envia documentos autorizados manualmente, prepara/revisa e registra atualizações de eventos confirmados e centraliza alterações técnicas recebidas por diferentes canais. As integrações Asana, Drive, sistema Legais e WhatsApp ficam para Fase 2. A credencial local do Asana foi usada somente em consultas GET de leitura. Nenhum dado foi escrito no Asana ou salvo como exportação no pacote. Remova a credencial do `.env` e revogue-a após esta revisão.

**Decisões técnicas (B1, 16/10):** runtime React+TS (Vite) padrão Skip Cloud + PocketBase; SQLite; migrations em `pocketbase/migrations/`; test runner Vitest; binding = API de collections + hooks server-side preservando o contrato C3; limite de provisionamento de role conforme C4. Detalhes em `06_notas/2026-10-16-registro-b1.md`.

**Matriz de acesso (F1-02, 20/10):** Admin=all; CS, Engenharia e Legais=assigned; Liderança=all somente leitura; recursos/ações conforme C4; escrita exige read + projects.read; um perfil por usuário; aprovação técnica exige designação por projeto/disciplina (regra P9 revisada em 16/10).

**Dicionário de campos (F1-01, 21/10):** mínimo obrigatório nome + cliente; demais campos do C2 opcionais; nenhum dado pessoal obrigatório. Registro em `06_notas/2026-10-21-registro-f1-01-dicionario.md`. P1/P2 "ainda não definido" — bloqueiam apenas a integração da Fase 2.

**Modelo relacional (F1-03, 21/10):** clients/projects com versionamento C2 (version inicial 1), FKs RESTRICT, unicidade de code (quando preenchido) e normalized_name, índices de consulta aprovados; provisionamento por migration versionada (padrão B1). Registro em `06_notas/2026-10-21-registro-f1-03-modelo-relacional.md`. DÚVIDA aberta: unicidade de normalized_name diverge da permissão de nomes iguais em SPEC-1-007 — o consultor decide por emenda antes da migration (F1-15).

**Próximas dependências:** F1-05 (shell/navegação) é a próxima candidata de implementação; F1-15 (formulários + migration) depende da DÚVIDA do normalized_name e de interface acessível; F1-19 planejável após runtime provisionado; F1-04/F1-14 aguardam B3 (autenticação corporativa — decisão Thórus + Engenharia). Nada implementado ainda.

- [Escopo](01_projeto/escopo-definitivo.md)
- [Tasks Fase 1](04_fase-atual/fase.md)
- [SPECs](04_fase-atual/specs/00-INDICE.md)
- [Rastreabilidade](04_fase-atual/matriz-de-rastreabilidade.md)
- [Questionário](03_documentos/06-Questionario-levantamento-cliente.md)
- [Registro B1](06_notas/2026-10-16-registro-b1.md)
