# Fase 1 — Tasks gerais

**Entrega:** aplicação interna utilizável manualmente, com banco de dados, interface completa para o MVP e contratos de integração preparados.

Na Fase 1 não haverá chamadas reais a Asana, Drive, Gemini/Calendar ou WhatsApp. A equipe cadastra e atualiza os dados pela interface. Ver [SPECs da Fase 1](../02-Specs/00-INDICE.md).

## Fase 1 — Base manual do sistema de gestão

| ID | Task | SPEC | Dono | Dependência | Critério de aceite | Evidência / parada |
|---|---|---|---|---|---|---|
| F1-01 | Confirmar dados do projeto, papéis, campos mínimos, status iniciais e projetos-piloto para cadastro manual | F1.1–F1.3 | Thórus + Produto | — | Dicionário inicial e amostra aprovados; dúvidas registradas sem bloquear campos opcionais | Decisões P1, P2, P5, P8 e P9 |
| F1-02 | Aprovar método de login, matriz de papéis e atribuição de projetos | F1.1 | Thórus Admin | — | Matriz aprovada para Admin, CS, Engenharia, Legais e Liderança | Decisões B3/B4 e P8 |
| F1-03 | Definir modelo relacional, migrações, índices e contratos de integração futura | F1.2 | Produto + Engenharia | F1-01 | Entidades, chaves, origem manual e campos externos documentados; nenhuma credencial/API real necessária | Diagrama/dicionário e revisão de compatibilidade |
| F1-04 | Implementar banco, migrações, autenticação, autorização e trilha de auditoria | F1.1,F1.2 | Engenharia | F1-02,F1-03 | Sessão, papéis, atribuições e dados persistem; acesso é validado no servidor e alterações auditadas | Evidências por papel; parar se regra de acesso indefinida |
| F1-05 | Construir shell da aplicação, sidebar, navegação responsiva e estados globais da interface | F1.3 | Engenharia + Design | F1-04 | Todas as páginas do escopo são navegáveis, com labels em português, foco/teclado e estados vazio/erro/carregamento | Roteiro visual revisado com usuários Thórus |
| F1-06 | Implementar dashboard, carteira, pesquisa, filtros, cadastro e edição manual de projetos | F1.3 | Engenharia | F1-05 | Criar, consultar, editar e arquivar projeto manualmente; filtros respeitam autorização | Cenários CRUD persistem após nova sessão |
| F1-07 | Implementar detalhe do projeto: tarefas/fases, definições e alterações, eventos legais, referências de documentos e histórico | F1.3,F1.4 | Engenharia | F1-06 | Seções e formulários definidos nas SPECs funcionam manualmente; histórico preserva versões e autoria | Um projeto amostral percorre os fluxos manuais |
| F1-08 | Implementar validações, confirmações, mensagens de erro, busca global autorizada, acessibilidade básica e auditoria operacional | F1.1,F1.3,F1.4 | Engenharia + Produto | F1-07 | Sem telas em branco; erros orientam correção; ações destrutivas/estado crítico pedem confirmação | Checklist UI/UX e trilha auditável |
| F1-09 | Preparar modelo de integração Asana/Drive sem conectar serviços | F1.2,F1.4 | Engenharia | F1-03,F1-07 | Campos externos/GIDs, mapeamentos, interfaces/adapters, estados de conexão e área de configuração previstos; nenhuma chamada externa ou token | Revisão de contrato e prova de isolamento por feature flag/desativada |
| F1-10 | Validar aceite manual da Fase 1, persistência, acesso, backup/restore e baseline | F1.1–F1.4 | Thórus + Engenharia | F1-08,F1-09 | Usuários executam fluxos sem planilha/API; amostra é conferida; pendências e baseline de localização registrados | Aprovação do MVP manual e lista de bloqueios para Fase 2 |
