# Escopo definitivo — Sistema de Gestão Integrada de Projetos Thórus

**Cliente:** Thórus Engenharia Ltda.  
**Versão:** 1.0 · 01/10/2026  
**Base:** escopo e mapeamento existentes do workspace; conversa de consultoria transcrita; captura enviada sobre notas do Gemini; modelo de ata da Thórus.  
**Estado:** escopo consolidado para orientar desenho e execução. Itens marcados como “a confirmar” não devem ser automatizados até validação com a Thórus.

## 1. Resultado desejado

Entregar, em incrementos demonstráveis, uma aplicação de acompanhamento de projetos de engenharia que dê à equipe interna uma visão única por projeto de cliente, empreendimentos, responsáveis, fases, entregas, aprovações legais, decisões técnicas, alterações e documentos. O sistema deve reduzir a procura manual entre Asana, Google Drive, conversas e planilhas e evitar que a Engenharia trabalhe com definição ultrapassada.

O sistema será utilizável manualmente desde a Fase 1, com banco de dados próprio para os registros operacionais do MVP. Asana permanece fonte operacional dos dados atualmente mantidos lá até a integração/reconciliação da Fase 2; Google Drive permanece repositório documental; WhatsApp permanece canal existente. Na Fase 1, a equipe cadastra e mantém registros pela interface; a estrutura para IDs e mapeamentos externos é preparada, mas nenhuma integração é conectada. O sistema não apaga nem reescreve fontes externas.

**Indicadores de sucesso para a implantação piloto:**

- Fase 1: equipe consegue cadastrar e operar manualmente os projetos-piloto no banco e interface. Fase 2: 100% do recorte Asana aprovado é conciliado com identificador estável e vínculo à origem.
- Cada alteração técnica registrada com autor, data, origem, estado e definição vigente consultável.
- Cada entrega e evento legal publicado ao cliente tem registro de origem, destinatário, horário e resultado do envio.
- Pelo menos 90% dos documentos amostrados no piloto abrem pelo vínculo registrado no projeto; os restantes aparecem em fila de correção.
- Reduzir em pelo menos 50% o tempo mediano de localizar o estado atual e a última alteração de um projeto, medido numa amostra cronometrada antes/depois. Linha de base e amostra serão medidas no piloto.
- Nenhuma mensagem interpretada por IA altera definição técnica, envia documento restrito ou declara aprovação sem confirmação humana até uma decisão posterior explícita da Thórus.

## 2. Problema e contexto confirmados

Projetos de instalações BIM duram até cerca de 18 meses. Dados de cliente, etapas, decisões, entregas e aprovações ficam espalhados por Asana, Drive, reuniões, WhatsApp, e-mail e planilhas/formulários. O portal/site de definições já substituiu parcialmente a planilha antiga, mas mudanças posteriores ainda chegam de modo informal e podem ficar fora do registro operacional.

Na reunião, Amanda descreveu um sistema central de projetos, acesso aos arquivos do Drive, registro de mudanças técnicas, acompanhamento e comunicação com cliente. Também citou um agente no WhatsApp que notifique eventos (por exemplo, parecer do bombeiro e aprovação) e responda pedidos de informação/documento com base no projeto. Asana foi apontado como integração desejada e a API foi indicada como caminho a investigar.

A captura enviada informa que já existe uma automação que percorre reuniões da agenda da empresa e ativa as notas do Gemini. Isso é capacidade atual a aproveitar. A reunião não comprovou API, mecanismo de exportação, permissão ou localização automática dessas notas; por isso a primeira entrega guarda vínculo/importação revisada e só automatiza ingestão após validação técnica.

## 3. Pessoas e permissões

| Perfil | Necessidade | Acesso previsto |
|---|---|---|
| Atendimento / CS | Carteira, pendências, reuniões, comunicação e histórico | Ler projetos atribuídos; registrar solicitações e comunicações; enviar após regra aprovada |
| Projetista / Engenharia | Consultar definições vigentes, mudanças e documentos do seu projeto | Ler projetos atribuídos; confirmar leitura; propor ou registrar atualização conforme função |
| Aprovações legais | Atualizar protocolo, pareceres e aprovações | Registrar evento legal e anexar/vincular evidência |
| Liderança | Visão de carteira, riscos, atrasos e indicadores | Leitura transversal e configuração autorizada |
| Cliente | Consultar projetos autorizados, registrar definições/pedidos e receber atualizações | Acesso apenas aos próprios projetos e conteúdo liberado |
| Administrador | Conectar integrações, papéis, listas e modelos | Configuração restrita, auditada; sem credenciais expostas na interface |

O mapeamento exato de usuários, grupos, visibilidade por empresa cliente e responsável que aprova mudança técnica precisa ser confirmado antes de ativar portal externo ou envio automático.

## 4. Escopo funcional

### 4.1 Cadastro e visão do projeto

- Lista e painel com código, cliente/construtora, empreendimento, município/UF, contato, responsáveis, escopo contratado, fase, datas principais, status, resumo e links externos.
- Filtros por cliente, responsável, fase, situação, marco legal, alteração pendente e prazo.
- Detalhe por projeto com cronologia unificada de decisões, arquivos, reuniões, entregas, pareceres e comunicações.
- Identificador externo do Asana preservado como chave de reconciliação, além do identificador interno.

### 4.2 Preparação e integração com Asana

- **Fase 1:** incluir IDs externos opcionais, origem do registro, mapeamento versionado e interfaces/adapters para integração futura. A equipe pode manter o sistema integralmente pela interface manual. Não autenticar, chamar API, importar ou sincronizar Asana nesta fase.
- **Fase 2:** após aceite do banco/MVP e validação de P1/P2, implementar leitura controlada dos projetos/tarefas/campos aprovados. Não assumir que custom fields, status e estrutura são iguais entre projetos.
- A integração futura deve fazer upsert idempotente por GID, guardar origem e timestamps, paginar, expor divergências e encaminhar conflito com valor manual para revisão, sem sobrescrita silenciosa.
- Somente leitura na primeira conexão real. Escrita de volta requer decisão posterior, campos autorizados e aprovação dos donos do processo.
- Usar OAuth 2.0 com escopos mínimos, tokens no backend/secret manager, desconexão e revogação. Confirmar recursos, scopes, paginação e limites na [API oficial Asana](https://developers.asana.com/reference/rest-api-reference) antes de implementar. Webhooks/reconciliação ficam para a fase de confiabilidade, não para Fase 1.

### 4.3 Google Drive e estrutura documental

- No cadastro do projeto, criar a pasta a partir do modelo aprovado pela Thórus e registrar URL/ID.
- A estrutura de referência enviada é: `01 ENTREGAS-APROVAÇÕES`, `02 TRABALHO`, `03 OBSOLETOS`, `04 ARQUIVOS EXTERNOS`, `05 MODELOS`, `06 CHECKLIST`.
- Exibir documentos e links autorizados dentro do sistema; upload feito no sistema deve ir para a pasta do projeto, manter metadados (nome, tipo, autor, data, link e categoria) e registrar falhas.
- Confirmar se o padrão informado é modelo da pasta de projeto (imagem sugere Drive compartilhado > 02_PMP > 0 - Padrão de pasta) e quais nomes/campos variam; não criar diretórios em local de produção sem validação do destino/escopo de acesso.
- Não duplicar binários sem necessidade nem alterar/excluir arquivos do Drive por ação implícita.

### 4.4 Definições técnicas e mudanças

- Apresentar definições por projeto e disciplina com opção, texto de apoio e observação, respeitando o conteúdo atual do portal Thórus.
- Registrar pedido/decisão com estado: rascunho, aguardando validação, aprovado, recusado, substituído ou cancelado; a taxonomia final será confirmada pela Thórus.
- Cada alteração guarda valor anterior e atual, autor, hora, canal/origem, motivo, anexos, aprovador e efeito/observação de escopo quando informados.
- Uma ata, transcrição ou mensagem pode originar sugestão de mudança. Conteúdo extraído por IA é rascunho; pessoa autorizada confirma a disciplina, o texto e o estado antes de se tornar definição vigente.
- Uma definição substituída permanece no histórico e não aparece como vigente. Mudança crítica gera alerta aos responsáveis somente após confirmação.

### 4.5 Reuniões, atas e decisões

- Associar reunião ao projeto, data, participantes, agenda, link da chamada e vínculo à nota/transcrição do Gemini existente.
- Modelo da ata fornecido: título/número do projeto, informações gerais (data, local, presentes) e itens definidos. A ata digital acrescenta decisões, pendências (responsável e prazo), riscos, origem e link para documento.
- Não transcrever/gravar reunião pelo novo sistema no piloto. Não presumir acesso programático às notas do Gemini: validar permissões e mecanismo de exportação; enquanto isso, usuário registra o link/arquivo e revisa as decisões extraídas.
- A ata só se torna registro aprovado depois de revisão humana; o sistema mantém o texto fonte e os itens estruturados separados.

### 4.6 Marcos, entregas e aprovações legais

- Registro estruturado de marcos por projeto: fase, entrega, postagem, protocolo, parecer, aprovação, exigência ou outro evento configurado.
- Incluir responsável, data, estado, observação e evidência/link; controlar histórico de mudanças.
- A comunicação ao cliente só pode ser disparada a partir de evento aprovado e modelo validado, com log do conteúdo, canal, destinatário e resultado.
- Regras de prazo, nomes oficiais dos marcos e quem tem autoridade para registrar/aprovar precisam ser confirmados no piloto.

### 4.7 Comunicação e agente de atendimento

- Alertas internos no sistema/e-mail conforme preferência e regras aprovadas.
- Relatório de status: escopo inicial considera automatizar a compilação dos dados oficiais e preparar o relatório. A periodicidade aparece quinzenal no escopo-base, enquanto a reunião anterior menciona relatório semanal; confirmar frequência, público, canal, campos e opt-out antes de ativar envio.
- WhatsApp: fase posterior, após identificar provedor/API oficial, número, consentimento, templates, política de retenção, roteamento e custos. O agente pode consultar apenas dados/documentos explicitamente liberados, responder com link, criar solicitação e encaminhar exceções a uma pessoa.
- Ações com efeito externo (confirmar alteração, prometer prazo, enviar arquivo restrito, comunicar aprovação) exigem confirmação humana. O agente registra o pedido, fonte consultada e resposta.

### 4.8 Busca e assistente contextual

- Busca por código, cliente, empreendimento, disciplina, data, etapa e texto de registros autorizados.
- Perguntas do usuário devem retornar resposta acompanhada de fonte/link e data, ou informar que não localizou evidência. A IA não preenche lacunas nem cria uma definição técnica.
- Primeiro escopo: consulta interna a dados/arquivos permitidos. Acesso do cliente e WhatsApp dependem da revisão de permissões e do piloto.

### 4.9 Interface e operação manual da Fase 1

- Sidebar: Dashboard, Projetos e Configurações (Configurações só para Admin); identidade/papel e Sair ao final. Abas de cada projeto: Visão geral, Tarefas e fases, Definições e alterações, Legais e marcos, Documentos, Atividade.
- Páginas: Entrar; Dashboard; lista/pesquisa de Projetos; Novo/Editar projeto; detalhe do projeto; configurações de usuários/papéis, listas de status/fases/disciplinas e mapeamentos externos planejados.
- Dashboard mostra contagens por fase/situação, tarefas vencidas/próximas, alterações aguardando validação, eventos legais recentes e atividade. Cada indicador abre a carteira já filtrada.
- Formulários permitem CRUD manual conforme papel para projetos, tarefas, fases, definições/versionamento, eventos legais e referências de documentos. Arquivamento preserva histórico; decisões/definições têm autor, data, estado e versão.
- Banco relacional armazena usuários/papéis/atribuições, clientes, projetos, fases/status, tarefas, definições e versões, eventos legais, referências documentais, auditoria, IDs externos opcionais e mapeamentos preparados.
- UI em português, responsiva, com busca/filtros/paginação, breadcrumb, estados vazio/erro/carregamento/sucesso, validação no servidor, confirmação de ações críticas, teclado/foco acessíveis e datas locais. Detalhamento: [SPEC F1.3 — Interface e operação manual](../02-Specs/spec-03-carteira-projetos.md).
- Referências documentais aceitam nome/categoria/URL manual; Fase 1 não cria pastas, faz upload/download ou valida ACL no Drive.
- Não há chamada de rede para Asana, Drive, Calendar/Gemini ou WhatsApp, botão de conexão, pedido de token nem automação externa na Fase 1.

## 5. Fluxo operacional alvo

1. **Fase 1:** Atendimento cria o projeto manualmente na aplicação, preenche equipe, escopo, fases e situação; banco persiste o registro e identifica autor/fonte manual.
2. **Fase 2:** após mapeamento aprovado, integra Asana e associa a pasta Drive validada. Divergências com cadastros manuais aparecem como pendência para conciliação, sem sobrescrita automática.
3. Cliente e equipe registram definições no canal escolhido; sistema mantém vigente + histórico.
4. Reunião acontece e notas do Gemini seguem o fluxo atual. Link/arquivo é associado ao projeto; decisão ou alteração é extraída como rascunho e validada por humano.
5. Engenharia consulta a definição vigente e registra/consulta entregas e marcos legais com evidência.
6. Mudanças e eventos aprovados atualizam cronologia, fila de responsáveis e, quando liberado, comunicação ao cliente.
7. Relatório apresenta dados atuais com fonte/data de atualização; erro de integração vira pendência visível.

## 6. Dados e origem

| Dado | Fonte inicial | Política |
|---|---|---|
| Identificador, projeto, tarefas, responsáveis, campos de acompanhamento | Fase 1: registro manual na aplicação; Fase 2: Asana para os campos acordados | Marcar origem e autor manual; depois vincular GID e conciliar; nunca sobrescrever silenciosamente |
| Dados do cliente, empreendimento, disciplina e decisões | Fase 1: registro manual aprovado; futuro: Asana e plataforma/site Thórus conforme decisão por campo | Fonte de verdade será decidida por campo em P2; manter histórico e conflito visível |
| Documentos, atas, plantas, ART e comprovantes | Fase 1: referências inseridas manualmente; Fase 2: Drive compartilhado | Guardar referência/ID e permissão quando integrada; cliente recebe somente itens explicitamente liberados |
| Reunião, agenda, participantes e link da nota | Google Calendar / automação atual do Gemini, a validar | Link e referência primeiro; conteúdo importado com revisão |
| Mensagens e pedidos | Entrada humana/WhatsApp futuro | Converter em pedido rastreável; origem e consentimento registrados |
| Eventos de status | Equipe de Engenharia/Legais/CS | Estado estruturado, autor e evidência obrigatórios |

Dados pessoais de contatos só são usados para a finalidade do projeto, com autorização e visibilidade por papel. Tokens e segredos ficam fora de documentos, logs e cliente web.

## 7. Fases de entrega

| Fase | Resultado visível | Capacidades incluídas | Gate de aceite |
|---|---|---|---|
| 1. MVP manual com banco e interface | Aplicação interna navegável e útil para cadastrar/acompanhar projetos sem dependência de integração | Login e papéis; banco relacional; dashboard; sidebar/páginas; CRUD manual de projetos, tarefas, definições, eventos legais e referências; auditoria; contratos de integração preparados | Usuário percorre fluxos manuais; dados persistem entre sessões; permissões, histórico, formulários e estados da UI aprovados; nenhuma chamada externa |
| 2. Integrações Asana e Drive | Registros conciliados com Asana e referências/arquivos Drive vinculados | OAuth e leitura Asana; mapeamento aprovado; upsert/reconciliação; associação de pastas Drive, referências e upload conforme ACL; conflitos manuais visíveis | Amostra Asana/Drive conciliada sem duplicatas; links/acesso aprovados; falhas reprocessáveis; registro manual não sobrescrito silenciosamente |
| 3. Reuniões, marcos e comunicações | Linha do tempo do projeto e relatório pronto para revisão | Associação da reunião/nota Gemini; ata baseada no modelo; decisões/pendências; marcos/legais; relatório de status; comunicações em modo rascunho/aprovação | Itens da amostra rastreiam à fonte; mensagem de teste permanece em rascunho até aprovada; relatório reflete campos e horário de atualização |
| 4. Portal e agente assistido | Cliente consulta projetos autorizados e registra pedidos; equipe atende fila | Acesso externo isolado por cliente; busca/documentos liberados; agente WhatsApp ou canal confirmado para FAQ e protocolo de pedidos; encaminhamento humano e auditoria | Testes de isolamento sem vazamento entre clientes; agente responde com fonte ou transfere; sem ação de escrita/envio sem aprovação; consentimento/canal configurados |
| 5. Operação integrada e confiabilidade | Fluxo ponta a ponta validado em piloto e preparado para ampliação | Habilitar sincronização de escrita se aprovada; webhooks + reconciliação; relatório no canal/frequência decididos; monitoramento, recuperação de falhas, auditoria e revisão de automações | Matriz ponta a ponta aprovada pelos donos; recuperação sem duplicar ações; nenhuma falha silenciosa; métricas do piloto medidas; plano de suporte/retorno testado |

Cada fase entrega sistema utilizável. Assistente/automação não substitui desenvolvimento do produto. Nas fases 4 e 5, agentes e rotinas automatizadas são capacidade adicional e continuam limitados a regras, fontes e aprovações especificadas.

## 8. Fora do escopo desta versão

- Substituir integralmente Asana, Drive, Google Calendar, plataforma de definições ou WhatsApp.
- Migração destrutiva, apagar dados externos, reescrever projetos históricos em massa ou enviar comunicações não aprovadas.
- Executar ou certificar projeto de engenharia, cálculo BIM/Revit, decisões de projeto, responsabilidade técnica ou aprovação legal.
- Gravar reuniões ou criar a automação Gemini existente de novo.
- Importar automaticamente toda conversa histórica de WhatsApp/Google Chat/e-mail.
- Atendimento autônomo que aprove alterações, assuma prazos, libere faturamento ou envie documentos sem controle.
- Publicar aplicação, configurar número pago/provedor, migrar para produção ou contratar serviços terceiros nesta etapa de especificação.

## 9. Premissas e decisões pendentes

As P1–P10 devem ser confirmadas com a Thórus usando o [questionário simples com instruções passo a passo](../05-Levantamento/06-Questionario-levantamento-cliente.md). Para a versão Word, use [DOCX editável](../05-Levantamento/06-Questionario-levantamento-cliente.docx). O questionário inclui passos para localizar projetos, campos, tarefas e status no Asana e descreve acesso de API sem pedir que tokens ou senhas sejam enviados.

| ID | Ponto a confirmar | Bloqueia |
|---|---|---|
| P1 | Quais projetos/campos/tarefas do Asana entram no piloto; workspace e owners | Mapeamento e integração |
| P2 | Fonte de verdade campo a campo; se sistema escreve no Asana e quais campos | Escrita bidirecional |
| P3 | Pasta raiz real, nomeação final com número/município/UF, modelo aprovado e permissões | Criação automática em Drive |
| P4 | Como acessar/exportar notas Gemini; agenda/calendário de serviço e política de dados | Ingestão automatizada |
| P5 | Modelo oficial de status/etapas legais, quem registra e quem aprova cada tipo de evento | Alertas e comunicações |
| P6 | Frequência de relatório: semanal na reunião anterior ou quinzenal no escopo-base; destinatários e conteúdo | Envio automático |
| P7 | WhatsApp: solução/provedor e API, número, opt-in, templates e custo | Agente e envio externo |
| P8 | Papéis e isolamento de clientes/contatos; conjunto de arquivos públicos para o cliente | Portal e busca externa |
| P9 | Quem pode confirmar mudança técnica por disciplina e como sinalizar impacto de escopo/prazo | Definição vigente |
| P10 | Base e amostra para medir tempo atual de localização e redução de retrabalho | Metas de resultado |

## 10. Integrações, confiabilidade e proteção

- Asana: REST API oficial, OAuth 2.0, escopos mínimos por endpoint, paginação, respeito a rate limits, tratamento de `401/403/429/5xx`, cursor/checkpoint, retries com backoff, idempotência e reconciliação. Webhook precisa handshake e validação da assinatura/segredo; manter polling de recuperação porque eventos são “at most once”. Ver documentação oficial em [API REST](https://developers.asana.com/reference/rest-api-reference), [OAuth](https://developers.asana.com/docs/oauth), [OAuth scopes](https://developers.asana.com/docs/oauth-scopes) e [criação de webhooks](https://developers.asana.com/reference/createwebhook).
- Google Drive/Calendar/Gemini: escolher API e escopos depois de descobrir tenant, política Workspace e permissões efetivas. Os documentos oficiais da Thórus permanecem com ACLs do Drive; o sistema não deve contornar permissões.
- WhatsApp: confirmar provedor autorizado e opt-in. Validar eventos recebidos, assinar callbacks conforme provedor, limitar anexos, armazenar logs e dar saída humana.
- Segredos nunca aparecem no navegador, markdown, ata, prompt, log ou histórico de erro. Minimizar retenção de conteúdo de reunião e conversa; registrar acesso e ação relevante.
- Toda automação de efeito externo terá chave de idempotência, estado de envio, reprocessamento controlado e registro do resultado.

## 11. Critérios globais de aceite

1. Usuário autorizado consulta e altera apenas dados conforme papel/atribuição.
2. Registros manuais mostram autor e data; na Fase 2 os registros integrados também mostrarão fonte e hora de sincronização.
3. Os fluxos manuais concluem sem API; integrações são liberadas somente na fase definida e não sobrescrevem dados sem reconciliação.
4. Alteração mantém histórico anterior, autor e estado claro.
5. Arquivo do cliente só pode ser aberto por quem tem autorização correspondente no Drive e no sistema.
6. Informação de IA exibe fonte e não pode alterar definição vigente sem confirmação.
7. Envio ao cliente só ocorre mediante regra, destinatário, evento e modelo aprovados; log recuperável.
8. Falhas de Asana/Drive/Calendar/WhatsApp aparecem numa fila ou painel operacional quando essas integrações forem habilitadas nas fases correspondentes.
9. A Thórus aprova campos, papéis, modelos, frequência e critérios piloto antes de habilitar escrita ou mensagens em produção.

## 12. Referências de origem

- Reunião de consultoria e materiais fornecidos pela Thórus.
- Modelo de ata e imagem da estrutura de pastas fornecidos pela Thórus.
- Documentação oficial da API Asana, referenciada na SPEC da Fase 2.
