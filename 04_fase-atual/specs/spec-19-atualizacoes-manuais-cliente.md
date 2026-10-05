# SPEC-1-019 — Atualizações revisadas e enviadas manualmente ao cliente

**Fase:** 1
**Status:** planejada
**Dono:** Engenharia de produto (execução) e Thórus (validação)
**Origem no escopo:** 4.6, 4.7 e 4.9; rota `/projetos/{id}/comunicacoes`
**Degrau:** compor/revisar/registrar dentro do sistema; transmissão fica fora, manual.

## Contexto e decisões fechadas

Amanda pediu avisar o cliente quando houver mudança de status, entrega, parecer do Bombeiro ou aprovação. Para cumprir o objetivo sem integração na Fase 1, o sistema prepara rascunho a partir de evento confirmado, exige revisão humana, permite copiar o texto e registrar que a equipe enviou pelo WhatsApp atual. Não há envio, consulta ou leitura automática.

## Perguntas de validação do cliente

| Perguntas | O que confirmar | Referência |
|---|---|---|
| P5/P6 | Eventos comunicáveis, campos obrigatórios, frequência, destinatários e pessoa que aprova antes do envio. | [Questionário P1–P10](../../03_documentos/06-Questionario-levantamento-cliente.md) |
| P7/P8 | WhatsApp/provedor atual, consentimento e quais papéis podem ver contato/documento. A conexão técnica fica na Fase 2. | [Questionário P1–P10](../../03_documentos/06-Questionario-levantamento-cliente.md) |

## Resultado observável

Após um parecer de Bombeiros registrado e confirmado no sistema, CS cria rascunho contendo projeto, tipo de evento, data e fonte; revisa, aprova o texto, copia, envia pelo WhatsApp usado atualmente e registra a ação com ator/destinatário/horário. Timeline não afirma entrega do WhatsApp.

## Limites e dependências

- **Inclui:** lista de eventos comunicáveis, preview e edição de mensagem, aprovação de conteúdo, cópia manual, confirmação manual de envio e histórico.
- **Fora:** API WhatsApp, disparo automático, bot, importação de conversa, templates remotos, entrega/recibo automático ou promessas de prazo inferidas.
- **Pré-condições:** evento real de status/entrega/legal está confirmado; P5/P6/P7/P8 e grants revisados; C3 idempotente.
- **Atores:** CS prepara e envia; aprovador interno revisa conforme regra P5/P6; Engenharia/Legais confirma evento de origem.
- **Risco/plano B:** evento incompleto/obsoleto ou destinatário sem vínculo impede aprovação; atualizar dado ou direcionar ao responsável.

## Página e comportamento detalhado

URL `/projetos/{id}/comunicacoes`; página também pode ser acessada pela aba Comunicações no projeto.

| Área | UI/UX |
|---|---|
| Lista | Projeto, tipo de evento, destinatário, autor, atualizado em, estado Draft/Aguardando revisão/Aprovado para envio manual/Enviado manualmente/Cancelado |
| Nova atualização | Selecionar evento confirmado da linha do tempo; contato vinculado; texto de mensagem; contexto/fonte somente leitura; ação Salvar rascunho |
| Revisão | Prévia completa, origem clicável, destinatário e canal; Editar, Aprovar texto, Cancelar. Aprovar texto não envia. |
| Após aprovação | Botão “Copiar mensagem”; aviso “Abra o WhatsApp e envie manualmente”. Ação “Registrar como enviada” pede confirmação. Nunca renderizar “Enviar pelo WhatsApp”. |
| Log de envio | Registra operador autenticado, contato, canal whatsapp_manual, instante, conteúdo final e versão da origem. Estado continua “Enviado manualmente”; sem confirmação de entrega. |
| Conflito | Se evento/fonte mudar após revisão, invalida aprovação e volta a “Revisão necessária”; texto não é enviado pelo sistema. |

## Dados de entrada e saída

| Campo | Regra |
|---|---|
| project_id/source_event_id/source_version | Evento pertence ao projeto visível, estado confirmado e versão exata |
| recipient_contact_id | Contato já associado ao projeto; mudança exige revalidação |
| message_body | 1–4000; editável; sem segredo, dados de terceiro ou informação não liberada |
| state | draft/review/approved_for_manual_send/manually_sent/cancelled |
| reviewed_by/at; manually_sent_by/at | Gerados por servidor e evento de confirmação, nunca inferidos de copiar texto |
| idempotency_key/expectedVersion | Obrigatórios para criar/logar/editar |

## Dados e integrações

Somente API local; evento fonte é `legal_events` ou evento de fase/status já persistido. Nenhuma chamada externa é feita pela aplicação.

| Origem/destino | Fonte | Contrato | Permissão | Erro |
|---|---|---|---|---|
| UI ↔ `customer_updates` | Banco local | Envelope C3 e expectedVersion | `customer_updates.read/create/update/review/record_manual_send`; C4 project scope | 401/403/404/409/422/503 |
| Atualização ↔ source event | `legal_events`/projeto | FK + source_version imutável | Exige read do evento/projeto | Evento apagado/alterado invalida revisão |
| Mutação ↔ activity_events | Mesma transação | hash/versão do texto e ator; sem segredos | Herda grant | Falha reverte mutação |

### Regras de negócio

1. Somente status/entrega/evento legal confirmado gera rascunho. Parecer do Bombeiro é tipo de marco legal configurado, não texto deduzido de chat.
2. Criar, aprovar texto e registrar envio são ações distintas; copiar texto não avança estado.
3. Uma única atualização ativa por evento, contato e versão. Repetição de submit com mesma idempotency key retorna registro existente.
4. Se evento, contato ou grant mudar entre aprovação e envio, bloquear log e exigir revisão.
5. Mensagem sempre identifica o fato confirmado e sua data; ausência de prazo impede incluir previsão.
6. “Enviado manualmente” é confirmação do funcionário, não prova de entrega/leitura do WhatsApp.
7. Fase 2 poderá adicionar integração assistida mantendo revisão humana e trilha.

## API local desta entrega

- `GET /api/projects/{id}/customer-updates?state=&page=`.
- `POST /api/projects/{id}/customer-updates/drafts` `{sourceEventId,recipientContactId,messageBody,idempotencyKey}`.
- `PATCH /api/customer-updates/{id}` `{messageBody,expectedVersion}`.
- `POST /api/customer-updates/{id}/approve-text` `{expectedVersion}`.
- `POST /api/customer-updates/{id}/record-manual-send` `{recipientContactId,channel:whatsapp_manual,expectedVersion}`; sem chamada de rede.
- `POST /api/customer-updates/{id}/cancel` `{reason,expectedVersion}`.

## Fluxo principal e erros

1. CS seleciona evento legal “Parecer emitido”, revisa fonte e contato vinculado e cria rascunho.
2. A pessoa autorizada aprova o conteúdo; CS copia e envia fora do sistema.
3. CS retorna e confirma envio manual; timeline grava a versão textual, ator, destinatário, canal e hora.
4. Se evento pendente/alterado, contato não autorizado, membership revogada, duplicate submit ou falha DB, bloquear registro e conservar rascunho.

| Cenário | Resultado | Recuperação |
|---|---|---|
| Principal | Rascunho, revisão e envio manual são estados distintos e persistidos | Reler histórico |
| Não autorizado | Sem grant, projeto/contato invisível ou evento não confirmado não permite copiar/aprovar | Solicitar correção à pessoa responsável |
| Concorrência/falha | Mudança de source_version retorna 409; sem sucesso falso | Recarregar evento, revisar novamente |

## Critérios de aceite

- [ ] **SPEC-1-019-CA-01:** Evento confirmado gera rascunho editável ligado ao projeto, contato e fonte; usuário revisa/aprova, copia e registra envio manual persistido após reload.
- [ ] **SPEC-1-019-CA-02:** Copiar não envia nem altera estado; confirmação manual deixa claro que não prova entrega; fonte alterada invalida revisão e repetição idempotente não duplica registro.
- [ ] **SPEC-1-019-CA-03:** Evento pendente, projeto/contato/grant inválido, conflito ou falha impede avanço e conserva rascunho; nenhuma chamada externa ocorre.

## TDD da SPEC

| Etapa | Prova | Ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | CA-01 e CA-03 com fixture C5 | Exercitar evento confirmado/pendente, copiar e registrar sem implementação | Falha esperada pela ausência da tela/regra | Estado e resposta |
| GREEN | Todos CA | Implementar ciclo rascunho-revisão-cópia-log e persistência | Todos critérios; sem egress | Screenshot, releitura e logs |
| REFACTOR | Todos CA | Reexecutar keyboard/mobile/permissão/conflito | Nenhum envio inesperado/duplicidade | Relatório por CA |

**Fixture:** projeto/evento/contato fictício; parecer legal pendente e confirmado; destination WhatsApp não real.
**Evidência:** texto fictício, source version, request/response redigidos e histórico.

## Instruções de execução para o Ethos

1. Ler contratos D1–D6/C1–C5 e SPEC-1-012/014.
2. Implementar somente interface e log local; copiar não deve abrir integração.
3. Nunca usar contato real em fixture, disparar mensagem ou registrar como entregue.
4. Testar evento alterado, grants, idempotência, erro e viewports; parar se destinatário/evento não estiver confirmado.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério binário | Subseção | Prova | Evidência | Pré-condições | Parada | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| F1-39 | Preparar, revisar e registrar atualização manual ao cliente | Thorus | SPEC-1-019 | Evento confirmado gera rascunho revisado e envio manual fica auditado, sem conexão | Página/fluxo manual | CA-01: rascunhar, revisar, copiar e registrar fixture | Source event, contato fictício, texto e activity | P5/P6/P7/P8 e grants | Parar sem evento/contato aprovado | Planejada |
| F1-41 | Evitar duplicidade e preservar origem da atualização | Thorus | SPEC-1-019 | Repetição não duplica; evento alterado invalida aprovação e nenhum egress ocorre | Regras/API | CA-02/03: repeat submit + source version stale | 409, estado preservado e verificação sem egress | F1-39, C3, SPEC-1-012 | Parar se app tentar enviar | Planejada |


## Emendas

| Data | Origem | Alteração | Motivo |
|---|---|---|---|
| 05/10/2026 | Pedido da Amanda e reunião de 05/10 | Atualizações manuais na Fase 1 | Entregar valor antes da conexão WhatsApp |
