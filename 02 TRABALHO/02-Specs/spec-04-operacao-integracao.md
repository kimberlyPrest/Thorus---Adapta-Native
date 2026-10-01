# SPEC F1.4 — Operação manual, administração e auditoria

**Fase:** 1 · **Atores:** Admin Thórus, CS, Engenharia, Legais e Liderança conforme permissão.  
**Resultado:** operação diária manual é completa e rastreável; configurações de integração estão preparadas, porém inativas.

## Operação manual

- Criar, consultar, editar e arquivar projetos pela interface; CRUD de tarefas, definições/solicitações, eventos legais e referências documentais conforme papel.
- Formulários mostram validações e salvamento; falha de banco não pode resultar em mensagem de sucesso.
- O usuário consegue ver qual dado foi preenchido manualmente, quem alterou e quando, sem depender de um ciclo externo.
- Estado de tarefa, aprovação, marco ou definição muda apenas por ação humana autorizada; estado final e histórico são consistentes.
- Definição técnica aprovada tem uma única versão vigente por projeto/disciplina/chave, salvo exceção explicitamente modelada; substituir a vigente gera nova versão e mantém a anterior.
- Referência de documento pode ser adicionada/editada manualmente. URL não comprova que arquivo existe nem que o destinatário tem acesso; avisar esse limite ao usuário.
- Exportações/backup, se oferecidos no ambiente da aplicação, exigem autorização e auditoria; não incluir tokens nem dados além do escopo autorizado.

## Administração

- Admin pode ativar/desativar usuários, ajustar papéis e atribuições e manter vocabulários/configurações de fase/status/disciplina/tipo de evento.
- Configuração histórica referenciada por registros não é apagada; valor desativado deixa de ser selecionável para novos registros, mas continua legível no histórico.
- Área de integração futura mostra estado **“Planejada — Fase 2”**, campos externos já mapeados e itens ainda a confirmar. Não deve mostrar “conectado/desconectado” enquanto OAuth não existe.
- Não disponibilizar token, controle de conectar/desconectar, botão de sincronizar, reprocessar, OAuth ou chamadas de diagnóstico na Fase 1.
- Os IDs externos podem ser preenchidos manualmente apenas para preparar reconciliação, com origem/autor visíveis; isso não consulta o serviço correspondente.

## Auditoria e histórico

Registrar: login/logout e acesso negado conforme suporte do IdP; mudança de usuário/papel/atribuição/configuração; criação/edição/arquivamento de projeto; criação/edição/conclusão de tarefa; solicitação, revisão, decisão e substituição de definição; mudança de evento legal; CRUD de referência documental.

Cada evento inclui ID/correlation ID, ator, ação, entidade/ID, timestamp UTC, resultado e resumo de campos alterados. Evitar payload completo, segredos e dados pessoais desnecessários. A tela de atividade mostra datas locais e respeita o acesso ao projeto. Evento de auditoria relevante não pode ser editado pelo usuário comum; retificação é um novo registro.

## Falhas, recuperação e dados

- Timeout/falha de gravação apresenta mensagem com tentativa recomendada e preserva entrada sempre que seguro.
- Conflito concorrente não sobrescreve silenciosamente; oferece recarregar ou comparar valores.
- Seed de demonstração é marcado como amostra e não se mistura a projetos reais sem indicação.
- Documentar procedimento de backup e restauração compatível com o banco/plataforma escolhida; registrar data da última verificação. Restauração não remove trilha que deva ser retida segundo política decidida.
- Na Fase 1, observabilidade concentra-se em disponibilidade da aplicação, erros de gravação, falhas de autenticação e integridade de migração. Não há painel/ciclo de sync.

## Contratos de integração preparados, sem ativação

Para Fase 2, manter documentados os recursos e campos candidatos de Asana, os IDs externos, o mapeamento versionado e a estratégia de upsert/reconciliação. Para Drive, reservar vínculo de pasta/documentos e categoria. O desenho futuro precisa tratar paginação, idempotência, permissões, autorização, erros e reconciliação; detalhes de API são especificados na SPEC de Asana da Fase 2. Na Fase 1, esses contratos são apenas documentação/configuração persistida; nenhuma rede externa é acessada e nenhum segredo é solicitado.

## Aceite verificável

- Criar e atualizar manualmente cada entidade prevista produz resultado consistente no banco, sem depender de serviço externo.
- Erros de gravação e conflitos são recuperáveis e não reportam uma operação como concluída sem persistência.
- A atividade apresenta autor, ação e instante para mudanças relevantes; acesso ao histórico segue a matriz de papel/projeto.
- Papel sem autorização não consegue efetuar ação sensível por interface nem endpoint.
- Tentar utilizar a área de integração não permite iniciar conexão, sync, upload ou envio externo na Fase 1.
- Backup/restore validado preserva registros, relacionamentos e histórico necessários ao aceite.

## Perguntas de validação do cliente

Consulte P1/P2 para integração futura, P5/P9 para estados/autorizações, P8 para perfis e P10 para medir resultado no [questionário de levantamento](../05-Levantamento/06-Questionario-levantamento-cliente.md). P1/P2 não autorizam chamadas externas durante a Fase 1.
