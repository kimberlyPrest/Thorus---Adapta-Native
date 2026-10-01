# Fase 1 — MVP interno manual com banco e interface

**Fase:** 1 · **Resultado:** aplicação interna que a equipe Thórus consegue usar no dia a dia, cadastrando e atualizando os dados manualmente. O banco relacional e os contratos de integração ficam preparados, mas serviços externos não são conectados nesta fase.

## Objetivo de negócio

Disponibilizar um lugar único para acompanhar a carteira e a operação interna dos projetos de engenharia. Ao fim da fase, CS, Engenharia, Legais e Liderança devem conseguir entrar, localizar um projeto, manter seu cadastro, registrar tarefas, decisões, eventos e referências documentais, e consultar o histórico sem depender de uma integração para concluir o fluxo.

## Inclui

- Aplicação web interna autenticada, interface em português, responsiva e com permissões por papel/projeto.
- Banco relacional com migrações, chaves, índices, validações, trilha de auditoria e dados de origem.
- Sidebar, dashboard, carteira, formulários manuais, detalhe de projeto com abas, configurações e estados de interface descritos na SPEC F1.3.
- CRUD manual para clientes/empreendimentos e projetos; tarefas/fases; definições técnicas e solicitações de alteração; eventos/aprovações legais; referências a documentos; histórico.
- Dados de demonstração/piloto inseridos manualmente e identificados como tal.
- Estrutura de persistência e contratos para Asana e Drive: IDs externos opcionais, fonte, mapeamentos, conectores/adapters, estados e pontos de extensão.
- Campos de URL para referências externas cadastradas manualmente; abrir o link respeita a autorização original do serviço.
- Operação simples de backup/restore e exportação controlada dos registros próprios da aplicação, se prevista na plataforma escolhida.

## Limites explícitos

- Não chamar APIs, autenticar, sincronizar nem fazer importação de Asana, Drive, Calendar/Gemini ou WhatsApp na Fase 1.
- Não criar pastas nem fazer upload/download de documentos pelo sistema. A aba Documentos guarda metadados e URLs informados manualmente.
- Não enviar mensagens, e-mails ou notificações externas automaticamente.
- Não exigir que o usuário abra Asana para conseguir cadastrar, atualizar ou concluir os fluxos manuais do MVP.
- Não transformar campos mapeados ou endpoints planejados em telas operacionais de integração ativa. A preparação técnica não pode disparar tráfego externo nem pedir tokens.
- O escopo do portal de cliente, agente WhatsApp, gravação/ingestão de reunião e automações permanece em fases posteriores.

## Páginas do MVP

1. **Entrar / sessão:** autenticação corporativa escolhida; estados de sessão expirada, acesso negado e recuperação conforme IdP.
2. **Dashboard:** contadores de projetos por fase/situação, tarefas atrasadas e próximas, alterações pendentes, eventos legais recentes e atividade recente. Contadores clicáveis aplicam filtros da carteira. Não enviar alertas externos.
3. **Projetos:** tabela paginada, busca, filtros e ordenação; ação “Novo projeto”; edição/arquivamento conforme papel. Colunas mínimas: código interno, cliente, empreendimento, município/UF, fase, situação, CS, responsável técnico, próxima data relevante e atualizado em.
4. **Projeto — Visão geral:** resumo, responsáveis, datas, escopo, riscos/observações, fase e situação; ações de editar, arquivar e registrar atividade conforme permissão.
5. **Projeto — Tarefas e fases:** fases configuradas, tarefas manuais com responsável, prazo, prioridade, estado, descrição e conclusão; filtros por estado/responsável/prazo e criação/edição.
6. **Projeto — Definições e alterações:** definições atuais por disciplina, histórico de versões, pedido de alteração, motivo, estado, autor, revisor e decisão manual. Valor aprovado só muda por ação autorizada.
7. **Projeto — Legais e marcos:** protocolos, exigências, pareceres, aprovações e outros eventos configurados; data, responsável, situação, observação e evidência/link; alterações ficam no histórico.
8. **Projeto — Documentos:** referências cadastradas manualmente com nome, categoria, URL, observação, data e visibilidade interna. O MVP não move nem armazena arquivo binário.
9. **Projeto — Atividade:** eventos de auditoria legíveis para aquele projeto, com ator, ação, data, origem manual e link para o registro quando autorizado.
10. **Configurações (Admin):** usuários/papéis/atribuições; listas de fase, situação, disciplina, prioridade e tipos de evento; visualização da preparação de integração (mapeamentos/IDs) sem controles de conexão ativos.

## Navegação / sidebar

- **Dashboard**
- **Projetos**
- **Configurações** (somente Admin)
- Identidade do usuário e **Sair** na área inferior; estado da sessão visível e acessível.
- Páginas de tarefas, definições, eventos legais, documentos e atividade ficam como abas/seções do detalhe do projeto, evitando menu global excessivo.
- Em largura estreita, sidebar recolhe para navegação acessível; manter indicação de seção atual, rótulos compreensíveis, retorno consistente e breadcrumbs `Projetos > [Projeto] > [Seção]`.

## Banco de dados lógico

Usar o banco relacional disponível/definido pela aplicação; escolha de produto/hosting depende da arquitetura do projeto. Modelo mínimo:

| Entidade | Campos/regras principais |
|---|---|
| `users` | ID local, ID estável do provedor de login, nome, e-mail permitido, ativo, criado/atualizado |
| `roles`, `user_roles` | Papel Admin, CS, Engenharia, Legais ou Liderança; atribuições nunca inferidas só pelo e-mail |
| `clients` | ID, nome e metadados estritamente necessários |
| `projects` | ID interno, código único quando informado, cliente, empreendimento, cidade/UF, escopo, fase, situação, datas, CS, responsável técnico, arquivado, autor e timestamps |
| `project_memberships` | Projeto, usuário/equipe, papel ou permissão, período/estado ativo; controle de visibilidade por projeto |
| `project_phases`, `project_statuses` | Valores configuráveis e ordenação; preservar referência ao valor utilizado no histórico |
| `tasks` | Projeto, fase, título, descrição, responsável, prioridade, prazo, estado, conclusão e timestamps |
| `technical_definitions` | Projeto, disciplina, chave/opção, valor vigente, observação, estado e versão atual |
| `definition_versions` | Definição, versão, valor anterior/novo, motivo, solicitante, decisor, estado, origem e data; append-only para histórico decisório |
| `legal_events` | Projeto, tipo, protocolo, estado, datas, responsável, observação e referência de evidência |
| `document_references` | Projeto, nome, categoria, URL, visibilidade, fonte informada, data e autor; sem binário na Fase 1 |
| `activity_events` | Ator, ação, entidade, ID, instante UTC, origem e campos alterados permitidos; não guardar payload sensível integral |
| `external_links` | Entidade interna, sistema externo, workspace/GID/permalink opcionais; unique por sistema + workspace + GID quando preenchido |
| `integration_mappings` | Sistema/recurso/campo externo, campo interno, transformação aprovada, versão, ativo; cadastrado como configuração, sem invocar conector |

IDs internos são estáveis; relacionamentos usam FKs; campos frequentemente filtrados têm índices; código do projeto e IDs externos têm unicidade no escopo definido. Guardar datas em UTC e exibir em `America/Sao_Paulo`. Exclusão operacional deve preferir arquivamento; não apagar silenciosamente histórico de decisão. Campos pessoais são minimizados. Credenciais/tokens não fazem parte dessas entidades.

## Fluxos manuais mínimos

1. Admin cria usuário/atribuição e configura listas permitidas.
2. CS cria projeto e vincula cliente, empreendimento, fase, estado e responsáveis.
3. Responsável adiciona/atualiza tarefas e prazos; dashboard e detalhe refletem os dados persistidos.
4. Engenharia registra definição e solicitação; papel autorizado aprova/recusa; histórico mantém versão anterior.
5. Legais registra protocolo/parecer/aprovação e URL/evidência; atividade exibe autoria e tempo.
6. CS registra referência de documento e atividade; abrir URL é ação manual do usuário.
7. Busca, filtros e dashboard operam sobre os dados do banco, sem depender de disponibilidade das ferramentas externas.

## Critérios de aceite da Fase 1

- Todos os fluxos manuais mínimos concluem pela interface e os dados continuam no banco após sair/entrar novamente.
- Usuário de cada papel só lista/consulta/edita o que a matriz autoriza, inclusive por chamada direta de rota/API.
- Navegação mostra todas as páginas previstas; cada lista/formulário implementa carregando, vazio, erro, sucesso, validação, permissão negada e confirmação quando pertinente.
- Alteração de definição e status mantém autor e histórico; exclusão de projeto não remove eventos relacionados.
- O dashboard deriva seus números dos dados salvos e liga cada indicador à lista correspondente.
- Links de documento são manuais; nenhuma chamada de integração ocorre na Fase 1, mesmo se houver GID/URL preenchido.
- As interfaces/campos de integração têm contrato e mapeamento revisável, mas não solicitam nem armazenam token nesta fase.
- A Thórus percorre um projeto-piloto manual de ponta a ponta e aprova a usabilidade, campos, papéis e limitações.

## Dependências e perguntas

- P1/P2: Asana e fonte dos dados orientam os IDs e campos de integração futura, sem bloquear o CRUD manual mínimo.
- P5/P9: vocabulário de fases/eventos e autoridade de aprovação define listas e permissões.
- P8: papéis internos e eventual visibilidade futura do cliente.
- P10: baseline do esforço atual para acompanhar o resultado.
- Método de login e ambiente de banco são decisões técnicas pendentes, não credenciais solicitadas à Thórus.

## Perguntas de validação do cliente

Consulte P1, P2, P5, P8, P9 e P10 no [questionário de levantamento](../../03_documentos/06-Questionario-levantamento-cliente.md). Em Fase 1, as perguntas de APIs servem para preparar o contrato e planejar a Fase 2; não autorizam conexão agora.

## SPECs detalhadas

- [Acesso interno e permissões](spec-01-acesso-interno.md)
- [Modelo de dados e preparação de integrações](spec-02-banco-integracoes.md)
- [Interface e operação manual da carteira](spec-03-carteira-projetos.md)
- [Fluxos manuais, administração e auditoria](spec-04-operacao-integracao.md)
