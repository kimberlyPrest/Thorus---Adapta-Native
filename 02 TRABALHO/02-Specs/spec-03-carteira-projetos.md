# SPEC F1.3 — Interface completa e operação manual de projetos

**Fase:** 1 · **Resultado:** equipe Thórus consegue operar o MVP pela interface e banco locais, sem depender de Asana/Drive.  
**Dependências:** F1.1 acesso; F1.2 banco e migrações.

## Mapa de navegação

### Sidebar global

1. **Dashboard** — resumo acionável da carteira.
2. **Projetos** — lista/pesquisa e criação de projeto.
3. **Configurações** — somente Admin: usuários, papéis, listas de estados e mapeamentos futuros.
4. Área inferior: nome/papel do usuário, status de sessão e **Sair**.

Não colocar itens de detalhe de projeto na sidebar global. Ao entrar em um projeto, mostrar breadcrumb `Projetos > [Código/Nome] > [Aba]` e navegação secundária nas abas abaixo.

### Páginas e comportamento

| Página | Conteúdo e ações manuais | Acesso inicial |
|---|---|---|
| Entrar | Login escolhido para organização; erro, sessão expirada, usuário desativado e sem autorização com próximo passo claro | Todos os usuários internos cadastrados |
| Dashboard | Quantidade de projetos por fase/situação; tarefas vencidas/próximas; definições aguardando validação; eventos legais recentes; atividade recente. Cada cartão navega à lista filtrada que o compõe | Conforme carteira permitida |
| Projetos | Tabela paginada, busca, filtro combinado, ordenação, “Novo projeto”, abrir, editar e arquivar conforme papel | CS/Admin/Liderança; demais por atribuição |
| Novo/Editar projeto | Formulário para código, cliente, empreendimento, município/UF, escopo, fase, situação, datas, CS, responsável técnico, observações e membros. Campos obrigatórios são explicitamente marcados e configuráveis | Admin/CS; liderança se aprovado |
| Projeto — Visão geral | Identificação, contatos internos necessários, escopo, responsáveis, fase/status, datas, riscos/observações, próximos prazos e atividade recente | Membro atribuído; Admin/Liderança conforme matriz |
| Projeto — Tarefas e fases | Fases ordenadas; criar/editar/concluir/reativar tarefa; responsável, prazo, prioridade, estado, descrição e observação. Exibir vencida/próxima sem enviar alertas externos | CS/Engenharia/Legais de acordo com atribuição |
| Projeto — Definições e alterações | Definições atuais por disciplina; iniciar solicitação com valor proposto, motivo, solicitante, origem e impacto informado; papel autorizado aprova/recusa/cancela; comparar versão atual e anterior | Engenharia; aprovadores definidos por papel |
| Projeto — Legais e marcos | Registrar protocolo, exigência, parecer, aprovação ou tipo configurado; estado, número, data, responsável, comentário e link/evidência. Mudança de estado guarda autor e data | Legais/CS; leitura dos demais autorizados |
| Projeto — Documentos | Lista de referências cadastradas manualmente: nome, categoria, URL, observação, data e visibilidade. Adicionar/editar/remover referência com confirmação; abrir link informado em nova aba | Membros do projeto; visibilidade interna |
| Projeto — Atividade | Linha do tempo do que ocorreu: entidade/registro, ação, autor, instante local e origem manual; filtrar por tipo/período | Leitura conforme projeto e sensibilidade |
| Configurações — Usuários e papéis | Ativar/desativar conta, papel e projetos atribuídos; alterações auditadas; não cadastrar senha do usuário no sistema se IdP externo for usado | Admin |
| Configurações — Listas | Manter valores permitidos de fases, situações, disciplinas, prioridades e tipos de evento; alterações não apagam o valor histórico já usado | Admin |
| Configurações — Integrações planejadas | Mostrar campos externos mapeados, revisão/pendências e aviso “Conexão prevista para Fase 2”; preparar GID, workspace e permalink, todos opcionais; sem ação “conectar”, token ou chamada de rede | Admin somente |

## Dashboard e carteira

- Contadores são calculados dos registros autorizados do banco na hora em que a página é consultada; apresentar “atualizado em” como horário da consulta, não como sincronização externa.
- Os cards de atrasos usam somente prazo presente e estado não concluído; zonas de data são definidas em São Paulo.
- Busca por código, projeto, empreendimento e cliente permitido; texto não pesquisa conteúdo de projeto sem autorização.
- Filtros: cliente, empreendimento, fase, situação, CS, responsável técnico, município/UF, tarefas vencidas, alterações pendentes, eventos legais e arquivado/ativo, conforme permissões.
- Lista oferece paginação, ordenação visível e opção de limpar todos os filtros. Não há resultados e nenhum projeto são estados diferentes.
- Ação de arquivar preserva o histórico e pede confirmação; não excluir definitivamente por ação cotidiana.

## Regras dos formulários/fluxos

- Validar obrigatoriedade, tamanho, formato de e-mail/URL, datas coerentes e existência de referência selecionada no servidor e na interface; explicar o campo sem jargão.
- Salvar informa sucesso apenas após confirmação de persistência. Em falha, preservar campos preenchidos para correção/reenvio.
- Em conflitos de edição, alertar que o registro mudou desde a abertura e permitir recarregar/comparar antes de substituir.
- Ao arquivar projeto, concluir/reatribuir tarefa, aprovar definição ou mudar estado legal, mostrar o efeito antes da confirmação. Mudança aprovada em definição cria nova versão.
- Toda ação manual significativa registra ator, timestamp UTC, tipo da ação, entidade e alteração mínima necessária; evitar gravar texto pessoal integral em logs genéricos.
- Documentos aceitam apenas metadados/link. A Fase 1 não faz upload, download, preview de binário, criação de pasta nem verificação automática da ACL do Drive.
- Nenhum registro pode ser criado/modificado automaticamente a partir de Asana/Gemini/WhatsApp nesta fase.

## UI/UX e acessibilidade

- Interface em português brasileiro; sidebar com ícone e texto, item ativo claro, rótulos persistentes em formulários e breadcrumbs de contexto.
- Desktop como superfície principal e adaptação para tablet/celular: tabelas viram cartões/lista com campos essenciais; ações não podem depender de hover.
- Hierarquia visual prioriza situação, responsável e próximo prazo. Usar badge com texto e cor, sem cor como único sinal.
- Estados obrigatórios em páginas de dados: carregando, vazio (com ação de criar quando permitido), sem resultado, erro recuperável, acesso negado e sucesso após gravar.
- Formulário divide campos em grupos curtos e mantém dados em erro. Ação primária única e secundárias claramente nomeadas; confirmação para ações com impacto.
- Acessível por teclado, foco visível, ordem lógica, contraste legível, nome acessível em controles e mensagens de validação associadas ao campo.
- Datas exibidas em formato brasileiro e fuso `America/Sao_Paulo`; timestamps sem hora na origem não ganham horário inventado.
- Conteúdo livre é tratado como texto e não executa HTML/script. URL é validada e identificada como externa antes de abrir.

## Critérios de aceite verificáveis

- Um usuário CS cria projeto, cadastra responsável e tarefa, depois sai/entra e encontra os dados persistidos.
- Engenharia cria solicitação de alteração; aprovador autorizado decide; visão geral mostra definição atual e atividade conserva a versão anterior.
- Legais registra evento e evidência URL; consegue corrigir uma informação mantendo rastreabilidade conforme regra aprovada.
- Dashboard e filtros levam à lista coerente e nunca incluem projetos fora do acesso.
- Cada página implementa seus estados vazio, erro, carregamento e sucesso; formulários apresentam validação compreensível.
- Sidebar e detalhe permanecem navegáveis por teclado e viewport estreita; estado atual não depende apenas de cor.
- Um link externo manual abre o endereço informado; sistema não acessa API de Asana/Drive.
- Usuário sem permissão não consegue ler/gravar via URL direta, endpoint ou ID de registro.

## Perguntas de validação do cliente

Consulte P1, P2, P5, P8, P9 e P10 no [questionário de levantamento](../05-Levantamento/06-Questionario-levantamento-cliente.md). P1/P2 informam dados e mapeamentos futuros; P5/P9 estados/aprovadores; P8 papéis; P10 baseline. Instruções de API são para preparar Fase 2, sem conectar serviços na Fase 1.
