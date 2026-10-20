# Changelog

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
- DÚVIDA: o C2/SPEC-1-011 modela um único `decided_by`/`decidedBy`, uma decisão por solicitação e a criação atômica da versão vigente no momento da aprovação. A regra confirmada exige duas aprovações obrigatórias (Engenharia + CS) e atualização automática após ambas. Consultor deve definir como representar e auditar as duas aprovações, transições/intermediário e o ponto atômico que cria a versão vigente. Não inferir nem implementar alternativa sem atualização contratual. [Resolvida em 2026-10-16 pela revisão da regra acima.]
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
