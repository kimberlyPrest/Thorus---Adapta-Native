# Changelog

## 2026-10-21 — F1-06 concluída (carteira com busca, filtros e paginação autorizados)

- Task F1-06 concluída: rota `/projetos` implementada no app Skip (v0.0.5 `91b5d57`, v0.0.6 `2208d5f`, v0.0.7 `7f71374`) sobre fixtures C5 locais (projetos A e B; CS atribuído somente a A; B arquivado como sentinela de vazamento) com autorização local de semântica C4 — o universo autorizado é filtrado antes de busca, filtros, contagem e paginação, e nenhum resultado oculto entra no total. QA da plataforma verde nas três versões (suite de testes sem runner no projeto, dívida mantida).
- Página conforme a SPEC-1-006: contador autorizado no header; busca normalizada com debounce 300 ms e Enter imediato; filtros de cliente, fase, situação, CS, responsável técnico e UF (opções derivadas só do universo autorizado); ver arquivados; limpar filtros; ordenação nome/atualizado/próximo prazo (sem prazo ao final); tabela D2 com cards no mobile; paginação 25/50 — tudo persistido na URL, com voltar/avançar restaurando filtros e página.
- Verificação automática no preview: Admin vê 1 projeto ativo e 2 com arquivados; busca `?q=vale` e filtro `?cs=Carlos+CS` preservados na URL; projeto B nunca aparece em lista, total ou filtro; "Limpar filtros" restaura a visão ativa. Teste humano aprovado pelo champion ("funcionou", 11:51). SPEC-1-006-CA-01 fechada; a validação formal de CA-02 fica para a F1-22 (dono Thorus).
- Dívidas marcadas: dados e autorização locais (fixtures C5) até a migration da F1-15 e o binding C3 real; ações por linha e CTA Novo projeto desabilitados (SPEC-1-008/F1-15); perfil "ver como" em store singleton compartilhado entre shell e páginas (removível quando /api/me existir).
- Próximas dependências: F1-22 (validação da carteira) liberada; F1-15 depende da DÚVIDA do normalized_name; F1-04/F1-14 aguardam B3.

## 2026-10-21 — F1-05 concluída (shell implementado e testado)

- 2026-10-21 · [Thórus Admin] · Task F1-05 concluída: shell com sidebar, breadcrumb e rotas Dashboard/Projetos implementado no app Skip (v0.0.4, `8ac2ed5`) — QA da plataforma verde (setup/staticAnalysis/build/integrações; suite de testes sem runner no projeto, dívida marcada); preview verificado por automação (navegação, breadcrumb, item ativo, estado vazio) e teste humano aprovado pelo champion ("funcionou"). Fixtures de identidade Admin/CS até /api/me existir (SPEC-1-002); dívidas marcadas no código (tokens D2 por classe arbitrária; Vitest pendente de instalação — package.json é preventAI na plataforma).
- Próximas dependências: F1-06 (carteira) é a próxima candidata; F1-15 depende da DÚVIDA do normalized_name; F1-04/F1-14 aguardam B3.

## 2026-10-21 — F1-03 concluída (modelo relacional aprovado) + DÚVIDA de unicidade

- Thórus aprovou o modelo relacional de clients/projects para migração (SPEC-1-007-CA-02), registrado em `06_notas/2026-10-21-registro-f1-03-modelo-relacional.md`: versionamento C2 confirmado (`version` inteiro inicial 1; `created_at`/`created_by`/`updated_at`); FKs de histórico com RESTRICT, sem cascade; unicidade de `projects.code` quando preenchido (trim/maiúscula, ≤30) e de `clients.normalized_name`; `projects.name` sem unicidade; índices de consulta por client_id, phase_option_id, status_option_id, archived_at e normalized_name; provisionamento por migration versionada em `pocketbase/migrations/` (padrão B1), sem alterar o app Skip fora de task autorizada.
- DÚVIDA: a unicidade de `normalized_name` aprovada no formulário diverge da SPEC-1-007, que permite cadastrar cliente novo com nome igual a existente (sugestão por normalização, sem deduplicação destrutiva). O consultor decide antes da migration da F1-15: manter UNIQUE (emenda de SPEC) ou índice sem unicidade (leitura literal da SPEC). Enquanto não resolvida, a F1-15 não implementa essa constraint.
- Nada foi implementado: documento de política técnica, sem migration ou código. Baseline do repositório executável verificado no Skip (projeto 56846, v0.0.3 `fe5cc94`, não publicado; somente coleção auth `users`; nenhuma migration; alteração pendente em `.skip.config.json` permanece intocada).

## 2026-10-21 — F1-01 concluída (dicionário de campos aprovado)

- Thórus aprovou o dicionário de campos do cadastro manual (SPEC-1-007-CA-01), registrado em `06_notas/2026-10-21-registro-f1-01-dicionario.md`: o mínimo obrigatório permanece nome + cliente (clients.name 2–160; projects.name 2–160 + client_id); nenhum campo adicional foi tornado obrigatório; nenhum dado pessoal (CPF/CNPJ/endereço/contato pessoal) é obrigatório. Dicionário completo com campo, tipo, obrigatoriedade, permissão e exemplo fictício no registro.
- P1 (projetos/campos do piloto) e P2 (fonte de verdade por campo) registradas formalmente como "ainda não definido": não bloqueiam a Fase 1 manual — P1 bloqueia apenas o mapeamento do piloto/integração e P2 decide a fonte de verdade da Fase 2. Operação manual sem conexão ao Asana confirmada; cópia de dados importados: manter até decisão posterior.
- Nada foi implementado: task de validação de política, sem código ou migration. F1-15 (formulários) e F1-03 (modelo relacional) passam a usar este dicionário como teto — o que não está nele não pode ser exigido sem emenda de produto.

## 2026-10-20 — F1-02 concluída (validação da matriz pela Thórus)

- Thórus Admin validou e assinou a matriz papel × ação × escopo (SPEC-1-004-CA-01), fechando a F1-02. Matriz aprovada: Admin=all; CS, Engenharia e Legais=assigned; Liderança=all somente leitura; recursos/ações conforme C4 (projects, tasks, definitions, legal_events, documents, activity, dashboard, admin_*, client_requests, customer_updates); regras de composição confirmadas (ausência de grant nega; escrita exige read + projects.read; um perfil por usuário; aprovação técnica exige designação por projeto/disciplina); casos permitido/negado definidos.
- Verificação automática (documental) passou: critérios conferidos contra SPEC-1-004, contrato C4 e registro B1 (06_notas/2026-10-16-registro-b1.md). Nenhuma implementação técnica ocorreu — F1-02 é validação de política.
- Aprendizado registrado em 06_notas/aprendizado-continuo/controle.md.
- Próximas dependências: F1-19 (implementar matriz) planejável após runtime provisionado; F1-04/F1-14 seguem bloqueadas por B3 (método de autenticação corporativa — decisão Thórus Admin + Engenharia).

## 2026-10-16 — Registro do B1 (decisões técnicas de fundação)

- Produto/Engenharia (Adapta) registrou o B1 do contrato C1 em `06_notas/2026-10-16-registro-b1.md`: runtime React+TS (Vite) no padrão Skip Cloud com backend PocketBase; banco SQLite embutido; migrações versionadas em `pocketbase/migrations/`; test runner Vitest; binding concreto = API de collections do PocketBase + hooks server-side preservando o envelope e os códigos de erro do C3; limite de provisionamento de role conforme o C4 (provisionamento Admin-only, um perfil por usuário no MVP, último Admin protegido, `access_state` pending/ready, `policy_version`, concessões/revogações em `activity_event`).
- **F1-02 liberada para validação pela Thórus** — P8 respondida e B1 registrado; a matriz papel × ação × escopo pode ser assinada (SPEC-1-004-CA-01). F1-19 planejável após F1-02 concluída e runtime provisionado.
- Nada foi implementado: nenhuma migration executada, nenhum código gravado. F1-04 e F1-14 seguem dependentes do B3 (método de autenticação corporativa — decisão de Thórus Admin + Engenharia) antes de contas reais.

## 2026-10-16 — Revisão da regra P9 (F1-02)

- Thórus Admin revisou a regra de aprovação técnica para eliminar a divergência com C2/SPEC-1-011: um único aprovador designado por projeto/disciplina (Engenharia ou CS), com CS apenas avisado da decisão. A exigência anterior de coaprovação obrigatória Engenharia+CS fica retirada; a definição vigente é criada atomicamente na decisão do aprovador designado, conforme SPEC-1-011.
- Demais respostas P9 de 2026-10-07 permanecem: CS registra o pedido; Engenharia analisa impacto técnico; a mudança só vale após aprovação; avisar CS e Engenharia; não encaminhar ao Comercial por impacto em prazo/preço; CS decide eventual impacto comercial.
- A DÚVIDA registrada em 2026-10-07 está resolvida sem emenda de SPEC: o contrato atual (C2 `change_requests.decided_by` único + `project_approvers` por projeto/disciplina) já representa a regra revisada. Nada foi implementado.
- B1 permanece decisão de Produto/Engenharia: runtime, banco, ferramenta de migration/test runner e limite de provisionamento de role ainda não registrados. [Resolvido em 2026-10-16 — ver entrada "Registro do B1" acima.]

## 2026-10-07 — Validação F1-02 (P8/P9)

- Thórus Admin aprovou a matriz-base C4, com a decisão específica para Liderança: `all` somente leitura. P8 confirmado: Admin=all; CS, Engenharia e Legais=assigned; Liderança=all/read-only. Todos os perfis internos podem consultar documentos marcados como liberados ao cliente; portal não será usado neste momento; categorias indicadas: ART, plantas, modelos, memoriais e documentos de aprovação do projeto; não há aprovação individual requerida para a liberação; CS valida o vínculo do contato.
- P9 confirmado: CS registra o pedido; Engenharia analisa impacto técnico; Engenharia e CS são coaprovadores obrigatórios (não basta qualquer um isoladamente); a definição vigente muda automaticamente após ambas as aprovações, e CS participa da aprovação; avisar CS e Engenharia; não encaminhar ao Comercial por impacto em prazo/preço; CS decide eventual impacto comercial. [Revisado em 2026-10-16: coaprovação substituída por aprovador único designado; CS apenas avisado.]
- DÚVIDA: o C2/SPEC-1-011 modela um único `decided_by`/`decidedBy`, uma decisão por solicitação e a criação atômica da versão vigente no momento da aprovação. A regra confirmada exige duas aprovações obrigatórias (Engenharia + CS) e atualização automática após ambas as aprovações. Consultor deve definir como representar e auditar as duas aprovações, transições/intermediário e o ponto atômico que cria a versão vigente. Não inferir nem implementar alternativa sem atualização contratual. [Resolvida em 2026-10-16 pela revisão da regra acima.]
- B1 permanece decisão de Produto/Engenharia: runtime, banco, ferramenta de migration/test runner e limite de provisionamento de role ainda não foram identificados no handoff. [Resolvido em 2026-10-16 — ver entrada "Registro do B1" acima.]

## 05/10/2026
- Escopo v1.2 atualizado com os três fluxos prioritários manuais na Fase 1; adicionadas SPECs verticais 018/019, tasks F1-38 a F1-41 e tasks de integração Legal/WhatsApp para Fase 2.

## 2026-10-01

- Corrigida a estrutura do pacote para o padrão de pasta de cliente do plugin Adapta.
- SPECs e tasks da Fase 1 alinhadas ao contrato do plugin e às quatro projeções de tasks.
- Fase 1 definida como interface e banco utilizáveis manualmente; integrações Asana/Drive preparadas e adiadas para Fase 2.

## 2026-10-05

- Escopo definitivo atualizado para apontar às 17 SPECs verticais da Fase 1.
- Tasks e matriz de rastreabilidade atualizadas com vínculos de aceite específicos e tarefas estáveis.
- SPECs divididas por rota/fluxo, incluindo administração de usuários, funções/permissões e listas; cada uma cobre interface, comportamento, dados, autorização, persistência e critérios de aceite.
- Perguntas P1–P10 vinculadas às SPECs relevantes e questionário alinhado: Asana permanece sem integração na Fase 1; conexão prevista para Fase 2.

## 2026-10-05 — Revisão após reunião

- Fase 1 passou a incluir operação manual dos três objetivos prioritários: atualizações ao cliente, pedidos/documentos e mudanças técnicas recebidas por WhatsApp/e-mail/reuniões.
- Incluídas SPEC-1-018/019 e tasks F1-38 a F1-41 para fila documental e atualizações manuais.
- A integração da Fase 2 passou a abranger Asana, Drive, webhook do sistema Legais e envio assistido por WhatsApp, condicionados a documentação, provedor e segurança aprovados.
- Credencial local usada somente em consultas de leitura; revogação e remoção do `.env` recomendadas após a revisão.
