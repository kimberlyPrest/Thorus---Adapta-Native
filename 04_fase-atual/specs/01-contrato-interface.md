# Contrato de interface — Fase 1

**Revisão:** 05/10/2026. Esta é a base de produto para o MVP interno; identidade visual oficial da Thórus será aplicada quando fornecida. Os tokens abaixo são uma decisão de implementação para evitar que cada rota invente seu próprio desenho.

## D1 — Layout e navegação

- Desktop 1440 × 900 como referência; conteúdo com largura máxima 1440 px, padding 32 px e sidebar 240 px. Header de página: breadcrumb, título, descrição curta e uma ação primária alinhada à direita.
- De 768 a 1023 px: sidebar recolhida de 72 px, ícones com nome acessível e tooltip acionável por foco; conteúdo com padding 24 px. Abaixo de 768 px: menu em drawer modal, header 56 px, padding 16 px, conteúdo em coluna única.
- Sidebar global: Dashboard `/dashboard`, Projetos `/projetos`, Solicitações `/solicitacoes` (grant client_requests.read), Configurações (somente Admin). Submenu Admin: Usuários, Perfis e permissões, Listas e Integrações planejadas. Identidade e Sair ficam ao final.
- Projeto: header persistente com código/nome, cliente, fase/situação e badge Arquivado quando aplicável. Abas: Visão geral, Tarefas e fases, Definições e alterações, Legais e marcos, Documentos, Atividade, Comunicações. Na aba Definições, navegação interna Vigentes / Solicitações. Abas usam links com URL própria e indicam a rota atual.
- Recarregar URL ou usar voltar/avançar preserva aba, página e filtros via query string. Não abrir uma segunda sidebar para cada projeto. Menu colapsado e abas cabem por rolagem horizontal identificável; ações principais permanecem disponíveis.

## D2 — Tokens e componentes

| Elemento | Decisão de desenho |
|---|---|
| Tipografia | Stack system-ui; corpo 14 px/20 px, título página 28 px/36 px, subtítulo 18 px/26 px, labels 14 px/20 px com peso 600 |
| Superfícies | Fundo #F8FAFC, painel #FFFFFF, texto #0F172A, secundário #475569, borda #CBD5E1 |
| Ação primária/foco | Primária #0F766E com texto branco; hover #115E59; foco visível 2 px #0F766E e offset 2 px |
| Estados | Erro #B91C1C; atenção #92400E; informação #1D4ED8; todo estado tem ícone/texto além de cor |
| Espaçamento | Escala 4/8/12/16/24/32 px; gap formulário 16 px; painéis separados por 24 px |
| Formas | Painel/input radius 8 px; botão 6 px; borda 1 px; sombra somente overlays, não toda célula |
| Controles | Input e botão pelo menos 40 px; área de toque 44 px no mobile; uma ação primária por bloco |
| Tabela | Header fixo dentro do painel; linha 48 px; nomes longos truncados com conteúdo completo acessível; menu por linha com rótulo “Ações de [nome]” |
| Drawer | Desktop 520 px, máximo viewport; mobile ocupa tela; título, fechar, conteúdo rolável e rodapé Salvar/Cancelar; retorno do foco ao abridor |
| Confirmação | Modal nomeia registro, consequência e ação: “Arquivar projeto”, “Desativar usuário”, “Aprovar alteração”; Escape cancela; foco inicial na opção segura |

## D3 — Estados de página e formulário

| Estado | Comportamento obrigatório |
|---|---|
| Carregamento inicial | Skeleton com a estrutura da página; região aria-busy; não mostrar “nenhum registro” antes da resposta |
| Base vazia | Título e uma orientação concreta; CTA de criação se o papel tem permissão |
| Filtro sem resultado | “Nenhum resultado para estes filtros”; botão Limpar filtros; manter filtros visíveis |
| Falha de leitura | Banner “Não foi possível carregar [recurso]”; Tentar novamente repete GET local; distinguir de lista vazia |
| Validação | Erro abaixo do campo, associado por aria-describedby; resumo de erros recebe foco; manter todos os valores |
| Salvando | Label “Salvando…”; bloquear duplo submit; Cancelar não dispara outra gravação |
| Sucesso | Confirmar só após commit; toast aria-live; atualizar registro e leitura correspondente, sem reiniciar filtro |
| Conflito 409 | “Este registro mudou enquanto você editava”; mostrar atual × rascunho em campos autorizados; Recarregar mantém rascunho em memória para revisão; sem botão Sobrescrever silenciosamente |
| Sessão/permissão | 401 leva a Entrar com returnTo interno; 403 mostra ação indisponível; 404 de objeto fora da atribuição não revela sua existência |

## D4 — Interação e acessibilidade

- Cada campo tem label persistente, obrigatório sinalizado em texto e ajuda quando houver regra. Placeholder não substitui label. Conteúdo livre renderiza como texto.
- Dialog/drawer prende foco, fecha por Escape e devolve foco; interação funciona por Tab/Shift+Tab/Enter/Espaço. Checkbox da matriz tem nome com recurso, ação e perfil.
- Tabelas abaixo de 768 px viram cards contendo identificação, estado, responsável, data relevante e ações. Nunca esconder a única ação de editar em hover.
- Formulário dirty pede confirmação ao mudar de rota. Cancelar descarta somente após confirmação; erro de rede mantém o rascunho em memória, sem gravar dados pessoais em localStorage.
- Texto crítico, foco e controles devem passar contraste WCAG AA; evidenciar 375/768/1440 px, teclado e zoom 200%. Esta checagem pertence a cada SPEC de rota.
- Datas de calendário `YYYY-MM-DD` exibem `dd/mm/aaaa` sem conversão de fuso; instantes UTC exibem data/hora em America/Sao_Paulo. Sem data exibir “Não informado”.

## D5 — Coerência de produto

- Coleções: paginação no servidor com 25 itens padrão e opção 50; contador e total apenas no universo autorizado. Busca por texto normalizado sem diferenciar caixa/acentos. Filtros ficam na URL.
- Ocultar ações não autorizadas e exigir a mesma política no servidor. Acesso de leitura sem edição continua utilizável.
- Integrações exibem “Planejada — Fase 2”. Referência externa abre em nova aba por clique manual. Nunca tratar URL salva como arquivo validado/compartilhado.
- Rotas futuras sem implementação ficam ausentes do menu; uma rota entregue inclui leitura e mutação previstas, banco, autorização, auditoria e estados. Não liberar páginas com botões sem função.


## D6 — Solicitações e atualizações manuais ao cliente

- `/solicitacoes` é fila interna. Registrar pedido é ação principal; filtros por projeto/estado/origem; detalhe mantém resumo e histórico.
- `/projetos/{id}/comunicacoes` só exibe eventos aprovados e rascunhos. Aviso fixo: “O texto é enviado manualmente fora do sistema”. Copiar não altera estado. Registro de envio requer confirmação de operador/destinatário.
- Referências documentais exibem o link cadastrado e instrução para conferir no Drive se está liberado àquele contato. A aplicação não certifica ACL.
- Não exibir portal do cliente, WhatsApp conectado, botão de envio API, estado “Entregue”, busca de conversas ou automação externa na Fase 1.
