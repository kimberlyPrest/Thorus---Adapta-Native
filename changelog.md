# Changelog

## 2026-10-07 — Validação F1-02 (P8/P9)

- Thórus Admin aprovou a matriz-base C4; P8 confirmado: Admin=all; CS, Engenharia, Legais e Liderança=assigned, sendo Liderança somente leitura. Equipe interna pode consultar documentos marcados como liberados ao cliente; portal não será usado neste momento; categorias indicadas: ART, plantas, modelos, memoriais e documentos de aprovação do projeto; não há aprovação individual requerida para a liberação; CS valida o vínculo do contato.
- P9 confirmado: CS registra o pedido; Engenharia analisa impacto técnico; Engenharia e CS aprovam juntos; CS foi indicado como responsável por atualizar a definição oficial; avisar CS e Engenharia; a mudança só passa a valer após aprovação; não encaminhar ao Comercial por impacto em prazo/preço; CS decide eventual impacto comercial.
- DÚVIDA: a aprovação conjunta Engenharia+CS e o papel de CS na atualização da definição vigente não cabem claramente ao contrato atual: C2/SPEC-1-011 modelam um único `decided_by`/`decidedBy` e uma única ação `definitions.decide` por solicitação, com a versão vigente criada atomicamente na aprovação. Consultor deve esclarecer como representar duas aprovações obrigatórias e se CS é aprovador (elegível/designado) ou apenas executor do registro, antes de finalizar F1-02/alterar SPECs. Nenhuma regra foi inferida nem implementada.
- B1 permanece decisão de Produto/Engenharia: runtime, banco, migration/test runner e limite de provisionamento de role ainda não foram identificados no handoff.

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
