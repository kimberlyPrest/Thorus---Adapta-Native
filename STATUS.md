# Status do projeto

**Data:** 16/10/2026<br>
**Estado:** B1 registrado (decisões técnicas de fundação); F1-02 liberada para validação pela Thórus.

Na Fase 1, a equipe registra e atende pedidos de clientes, envia documentos autorizados manualmente, prepara/revisa e registra atualizações de eventos confirmados e centraliza alterações técnicas recebidas por diferentes canais. As integrações Asana, Drive, sistema Legais e WhatsApp ficam para Fase 2. A credencial local do Asana foi usada somente em consultas GET de leitura. Nenhum dado foi escrito no Asana ou salvo como exportação no pacote. Remova a credencial do `.env` e revogue-a após esta revisão.

**Decisões técnicas (B1, 16/10):** runtime React+TS (Vite) padrão Skip Cloud + PocketBase; SQLite; migrations em `pocketbase/migrations/`; test runner Vitest; binding = API de collections + hooks server-side preservando o contrato C3; limite de provisionamento de role conforme C4. Detalhes em `06_notas/2026-10-16-registro-b1.md`. Nada implementado ainda; F1-04/F1-14 aguardam B3 (autenticação corporativa — decisão Thórus + Engenharia).

- [Escopo](01_projeto/escopo-definitivo.md)
- [Tasks Fase 1](04_fase-atual/fase.md)
- [SPECs](04_fase-atual/specs/00-INDICE.md)
- [Rastreabilidade](04_fase-atual/matriz-de-rastreabilidade.md)
- [Questionário](03_documentos/06-Questionario-levantamento-cliente.md)
- [Registro B1](06_notas/2026-10-16-registro-b1.md)
