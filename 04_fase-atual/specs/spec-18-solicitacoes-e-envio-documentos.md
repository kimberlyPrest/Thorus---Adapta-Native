# SPEC-1-018 — Solicitações de clientes e envio manual de referências autorizadas

**Fase:** 1
**Status:** planejada
**Dono:** Engenharia de produto (execução) e Thórus (validação)
**Origem no escopo:** 4.3, 4.7 e 4.9; rota `/solicitacoes`
**Degrau:** sistema interno utilizável manualmente; nenhuma integração externa.

## Contexto e decisões fechadas

- A equipe hoje recebe pedidos por WhatsApp, e-mail e conversas/reuniões e localiza materiais no Drive.
- Na Fase 1, funcionário Thórus registra o pedido no sistema. O cliente não precisa de portal/login.
- A aplicação guarda referências e histórico, mas não baixa, envia, compartilha ou muda permissões de documentos.

## Perguntas de validação do cliente

| Perguntas | O que confirmar | Referência |
|---|---|---|
| P3 e P8 | Caminho do Drive, tipos de documentos que podem ser enviados e como a equipe confirma que o cliente pode abrir cada item. | [Questionário P1–P10](../../03_documentos/06-Questionario-levantamento-cliente.md) |
| P7 | Canal que a equipe usa hoje e quando uma solicitação precisa ser atendida por pessoa. | [Questionário P1–P10](../../03_documentos/06-Questionario-levantamento-cliente.md) |

## Resultado observável

CS registra o pedido de ART recebido no WhatsApp, associa-o ao projeto, encontra a referência da ART na lista de documentos, confirma manualmente a ACL no Drive, envia link/arquivo manualmente fora do aplicativo pelo WhatsApp existente e registra canal, documento, operador e data. A equipe encontra o atendimento após nova sessão.

## Limites e dependências

- **Inclui:** fila interna, formulário de registro, atribuição, detalhe/histórico, consulta de referências de documento, fechamento/cancelamento e registro de envio manual.
- **Não inclui:** portal do cliente, agente/IA, leitura de mensagens, API WhatsApp, API Drive, download, upload, compartilhamento ou alteração de ACL.
- **Pré-condições:** C2–C5, D1–D6, usuário autenticado `access_state=ready`, projeto atribuído e grants `client_requests.*` e `documents.read`.
- **Atores:** CS registra/atribui/conclui; Admin gerencia acesso; Engenharia/Legais consultam ou atualizam conforme grants. Cliente não acessa esta tela na Fase 1.
- **Risco/plano B:** contato/projeto ou permissão documental ambígua deixa o pedido “Aguardando confirmação”; não sugerir nem enviar documento.
- **Rollback:** cancelar encerra fluxo sem apagar request, comentário ou histórico.

## Página e comportamento detalhado

URL `/solicitacoes`; sidebar mostra a opção só com grant `client_requests.read`.

| Área | UI/UX e comportamento |
|---|---|
| Header | Título “Solicitações”, descrição “Pedidos recebidos pelos canais da equipe”, CTA “Registrar pedido” |
| Busca/filtros | Projeto, estado, origem, tipo, responsável, período; query string preserva filtro e paginação |
| Tabela | ID curto, projeto, solicitante, resumo, origem, tipo (informação/documento/alteração), responsável, estado, última atualização |
| Registro | Projeto obrigatório; pessoa/contato ou “não identificado”; origem WhatsApp/e-mail/telefone/reunião/presencial; tipo; resumo; referência/link opcional; responsável; não colar conversa inteira |
| Detalhe | Linha do tempo, resumo, origem, atribuição e referências do projeto; link “Abrir documento” em nova aba; aviso explícito “confira no Drive se pode compartilhar com este destinatário” |
| Envio manual | Campos de log: documento/referência, canal, destinatário, operador e data. A ação apenas registra o que a pessoa confirma ter feito fora do app. Não há botão que envie. |
| Estados | Novo, Em atendimento, Aguardando confirmação, Aguardando cliente, Concluído, Cancelado. Estados vazios/erro/loading seguem D3. |

## Dados de entrada e saída

| Campo | Regra |
|---|---|
| project_id | Obrigatório e visível ao ator conforme C4 |
| requester_contact_id/requester_label | ID preferido; label curta se contato ainda não cadastrado; não exigir telefone/e-mail pessoal |
| intake_channel | whatsapp/email/phone/meeting/in_person |
| request_type | info/document/technical_change/other; technical_change deve gerar/ligar a SPEC-1-011 |
| summary | 1–2000 caracteres, síntese do operador; não armazenar conversa integral ou segredo |
| source_reference | URL HTTP(S) opcional ou referência à ata/e-mail; validação C3; sem fetch automático |
| assignee_id/state/due_date | Responsável da equipe; enum; data calendário DATE opcional |
| sent_document_id/channel/recipient/operator/time | Preenchidos somente ao registrar atendimento manual; não significa entrega confirmada pelo serviço externo |

## Dados e integrações

Somente API local C3 e banco C2. As referências vêm de `document_references` da SPEC-1-013, limitadas a projeto autorizado. Nenhuma API Asana/Drive/WhatsApp/e-mail é chamada.

| Origem/destino | Fonte | Contrato | Autorização | Erro |
|---|---|---|---|---|
| Tela ↔ API local ↔ `client_requests` | Banco da aplicação | Envelope C3, expectedVersion, idempotência | `client_requests.read/create/update/assign/close` + escopo C4 | 401/403/404/409/422/503 sem vazar existência |
| Tela ↔ referências | `document_references` | Apenas metadados e link manual, sem abrir binário | `documents.read` e membership ativa | ACL externa não verificável pelo app; interromper e encaminhar |
| Mutação ↔ activity_events | Mesma transação | Ator, ação, request, documento referenciado, canal e instante | Herda grant | Falha no log reverte a mutação |

### Regras de negócio e dados

1. Busca e contagem restringem projetos antes de query/paginação; ID direto não concede acesso.
2. Link cadastrado não significa que o contato pode abrir. A equipe abre o destino e confirma a permissão fora do app antes de enviar.
3. A aplicação não baixa, anexa, envia, expõe publicamente, muda compartilhamento ou valida ACL Drive por HEAD/fetch.
4. “Registrar envio manual” exige documento, canal, destinatário, operador autenticado e confirmação. O estado final chama-se “Atendido manualmente”, nunca “Entregue”.
5. Alterações técnicas são convertidas em solicitação da SPEC-1-011 com origem e vínculo; a fila não altera definição vigente.
6. Uma falha mantém o formulário; fechamento e cancelamento são versionados e auditáveis.

## API local desta entrega

- `GET /api/client-requests?projectId=&state=&channel=&type=&q=&page=` lista apenas projetos autorizados.
- `POST /api/client-requests` cria com projeto, contato/label, origem, tipo, resumo, source reference, responsável.
- `PATCH /api/client-requests/{id}` edita resumo/responsável/estado mediante `expectedVersion`.
- `GET /api/projects/{id}/documents` reutiliza SPEC-1-013 e grant `documents.read`.
- `POST /api/client-requests/{id}/record-manual-send` registra referência, destinatário, canal, operador e versão esperada; não contata serviço externo.
- `POST /api/client-requests/{id}/cancel` requer motivo e versão.

## Fluxo principal e caminhos de erro

1. CS seleciona “Registrar pedido”, escolhe projeto, canal WhatsApp e tipo Documento, escreve resumo “Solicitou ART” e atribui responsável.
2. Responsável abre o pedido, localiza referência de ART do projeto e confirma ACL no Drive para aquele contato.
3. Responsável faz o envio pelo canal habitual fora do app, volta e registra item/canal/destinatário; pedido vira Atendido manualmente.
4. Recarga e busca exibem o mesmo histórico para membros autorizados; pessoa fora do escopo recebe 404/403 sem metadados.

| Cenário | Condição | Resultado | Recuperação |
|---|---|---|---|
| Principal | Pedido e documento autorizados | Criado, atendido e persistido com auditoria | Reler após reload |
| Permissão | Projeto invisível, grant ausente ou membership revogada | Nada retorna/nada grava | Solicitar atribuição ao Admin |
| Limite/erro | Projeto ambíguo, ACL não confirmada, link inválido, 409 ou falha DB | Não fecha como atendido; mantém rascunho | Corrigir vínculo ou encaminhar para confirmação humana |

## Critérios de aceite

- [ ] **SPEC-1-018-CA-01:** Pedido recebido fora do app fica rastreável por projeto, contato/label, canal, tipo, resumo, responsável e estado após recarga.
- [ ] **SPEC-1-018-CA-02:** Operador localiza referência apenas em projeto autorizado, confirma ACL externamente e registra envio manual com documento, canal, destinatário e ator sem afirmar entrega.
- [ ] **SPEC-1-018-CA-03:** Projeto/contato ambíguo, acesso não confirmado, usuário não autorizado, conflito ou falha impede conclusão e preserva rascunho; nenhuma chamada externa ocorre.

## TDD da SPEC

| Etapa | Prova | Ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | CA-01/CA-03 com fixture C5 | Tentar registrar, filtrar e concluir antes da implementação | Ausência reproduzível sem fixture inválida | Estado inicial e falha |
| GREEN | CA-01–03 | Construir fila/formulário/API/db/auditoria; testar grant e releitura | Todos CA passam; envio continua externo/manual | Capturas, resposta C3 e releitura |
| REFACTOR | Todos CA | Repetir teclado, mobile, conflito e usuário fora do escopo | Sem regressão, vazamento ou egress | Relatório e evidências por CA |

**Fixture:** pedido fictício de ART, Projeto A, referência Drive fake e CS atribuído; Projeto B como sentinela.
**Evidência:** screenshots sem dados reais, request/response redigidos, releitura e trilha.

## Instruções de execução para o Ethos

1. Ler contratos D1–D6/C1–C5 e SPEC-1-008/013/014.
2. Criar entidades/rota local; somente referências, sem binário nem API Drive.
3. Não criar acesso de cliente ou enviar mensagem/arquivo por automação.
4. Implementar principal, autorização/escopo, erro/409, teclado e mobile; anexar prova.
5. Parar se autorização Drive ou destinatário não puder ser confirmada.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério binário | Subseção | Prova | Evidência | Pré-condições | Parada | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| F1-38 | Registrar e atribuir pedidos de informação/documento recebidos por canais externos | Thorus | SPEC-1-018 | Pedido com projeto/origem/tipo/responsável persiste e é recuperado após recarga | Página e comportamento; API local | CA-01: criar WhatsApp fictício e reler | Fila, detalhe e activity event | C4 e fixtures fictícias | Parar em projeto/contato ambíguo | Planejada |
| F1-40 | Localizar referência autorizada e registrar atendimento documental manual | Thorus | SPEC-1-018 | Operador confirma ACL fora do app e registra referência/canal/destinatário/ator | Regras; detalhe da solicitação | CA-02: consulta, checagem humana e conclusão | Log do pedido e link fictício | SPEC-1-013, P3/P8 | Parar se ACL ambígua | Planejada |


## Emendas

| Data | Origem | Alteração | Motivo |
|---|---|---|---|
| 05/10/2026 | Solicitação da Amanda e reunião de 05/10 | Atendimento manual completo na Fase 1 | Integrar somente na Fase 2 |
