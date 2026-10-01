# SPEC F1.4 — Operação manual, administração e auditoria

**Fase:** 1<br>
**Status:** planejada<br>
**Dono:** Engenharia de produto<br>
**Origem no escopo:** seções 4.4, 4.6, 4.9 e 6; P1/P2/P5/P8/P9/P10<br>
**Degrau da solução:** capacidade operacional nativa do MVP; cadastros e decisões são explícitos e manuais.

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

## Critérios de aceite

- [ ] **CA-F1.4-01:** Criar e atualizar manualmente cada entidade prevista produz resultado consistente no banco, sem depender de serviço externo.
- [ ] **CA-F1.4-02:** Erros de gravação e conflitos são recuperáveis e não reportam uma operação como concluída sem persistência.
- [ ] **CA-F1.4-03:** A atividade apresenta autor, ação e instante para mudanças relevantes; acesso ao histórico segue a matriz de papel/projeto.
- [ ] **CA-F1.4-04:** Papel sem autorização não consegue efetuar ação sensível por interface nem endpoint.
- [ ] **CA-F1.4-05:** Tentar utilizar a área de integração não permite iniciar conexão, sync, upload ou envio externo na Fase 1.
- [ ] **CA-F1.4-06:** Backup/restore validado preserva registros, relacionamentos e histórico necessários ao aceite.

## Perguntas de validação do cliente

Consulte P1/P2 para integração futura, P5/P9 para estados/autorizações, P8 para perfis e P10 para medir resultado no [questionário de levantamento](../../03_documentos/06-Questionario-levantamento-cliente.md). P1/P2 não autorizam chamadas externas durante a Fase 1.


## Contexto e decisões fechadas

- **Estado atual:** o fluxo futuro de Asana/Drive ainda não está conectado; o primeiro ciclo de trabalho depende de operação manual.
- **Estado desejado:** operação CRUD interna administrável e auditável; Configurações apresenta a integração como planejada, não conectada.
- **Decisões já fechadas:** nenhum token/OAuth/sync/upload/envio na Fase 1; atividade de negócio é manual; arquivar não apaga histórico; definições legais/técnicas só mudam por papel permitido.
- **Bloqueios:** retenção e papéis P8/P9 podem limitar dados, mas não impedem operar registro manual mínimo após validação dos campos necessários.

## Resultado observável

Admin mantém usuários/listas; CS, Engenharia e Legais registram eventos por papel; histórico identifica ator/hora/alteração; painel de integrações informa “Planejada — Fase 2” e não dispara chamadas externas.

## Limites e dependências

- **Inclui:** administração de usuários/roles/lists, auditoria de ações manuais, mensagens de gravação/recuperação e preparação inativa dos conectores.
- **Fora de escopo:** OAuth, sync, reprocessamento, webhook, upload, notificação a cliente ou integração automática.
- **Entradas/pré-condições:** papéis aprovados, modelo F1.2, regras de aprovação P5/P9 e retenção definida para logs/backup.
- **Saídas:** painel de operação local, activity events e configuração de integração marcada inativa.
- **Atores:** Admin altera usuários/listas; cada área atualiza seu tipo de registro; somente aprovador definido altera definição vigente.
- **Superfícies:** Configurações, Activity do projeto, formulários, log/auditoria e procedimento de backup/restore.
- **Risco/plano B:** erro no serviço local/database bloqueia gravação; exibir status e manter formulário para retry, sem fallback para planilha automática.
- **Rollback:** desativar configuração/feature de integração; preservar registros; corrigir estado por novo evento, não apagar auditoria.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Usuário → banco/auditoria | Banco interno | actor, action, entity, entity_id, timestamp UTC, outcome, diff mínimo | Papel/projeto F1.1 | Mutação e audit event atômicos quando possível | Se persistência falhar, não apresentar sucesso; retry não duplica decisão |
| UI Configurações → conectores | Estado sempre inativo em Fase 1 | system, resource, mapping_version, label “Planejada — Fase 2” | Admin pode revisar mapping; sem segredo | Nenhuma chamada de rede | Botões/API de conector ausentes ou desabilitados e bloqueados no servidor |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-F1.4-01 | Configuração/lista tem registro histórico | Desativar remove da seleção nova, preserva leitura histórica | Nunca apagar opção referenciada | SPEC F1.4 administração |
| RN-F1.4-02 | Usuário tenta abrir conexão na Fase 1 | Não há ação executável nem chamada externa | Admin vê previsão de Fase 2 | Limites explícitos F1 |
| RN-F1.4-03 | Registro sofre correção de decisão relevante | Nova versão/evento registra antes/depois, ator e motivo | Texto sensível deve ser minimizado no log | SPEC F1.4 auditoria |

## Fluxo e regras

1. Admin ativa/desativa conta, papel/atribuição ou opção de lista; alteração e instante são auditados.
2. Usuário registra uma mudança de negócio; servidor valida permissão e persiste o registro/histórico juntos.
3. Em erro de gravação, UI conserva entrada e explica retry; nunca mostra estado concluído sem confirmação de banco.
4. A tela de integração futura mostra contrato/mapeamento e indicação inativa; backend rejeita endpoint que tente iniciar conexão.
5. Backup/restore em ambiente de teste confirma relações e histórico necessários.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Admin muda status/lista e usuário registra ação manual | Alteração salva e aparece na auditoria de projeto/configuração | Acesso negado informa contato Admin |
| Limite | Usuário tenta usar integração planejada | Nenhum controle de conexão disponível; backend não inicia ciclo | Exibir que conexão é da Fase 2 |
| Falha | Banco falha durante alteração/auditoria | Transação não produz sucesso parcial | Preservar formulário; restaurar banco; repetir sem duplicar |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** matrizes de papel, F1.2, regras RN-F1.4-01–03 e retenção aprovada.
2. **Alterar somente:** configuração manual, auditoria, mensagens e estado inativo de conector.
3. **Não alterar:** nenhuma integração real, side effect externo, retenção não aprovada ou autorização de cliente.
4. **Executar nesta ordem:** CRUD de configuração; auditoria; recuperação; prova de conector desligado; backup/restore.
5. **Parar quando:** log exigiria conteúdo pessoal integral, alteração apagaria histórico ou tentativa de conexão exigir credencial não prevista.
6. **Estado válido ao parar:** operação manual disponível, auditoria coerente e serviços externos inalcançados.

## Checklist de execução

- [ ] Papéis de alteração de usuário/listas e decisões definidos.
- [ ] Ações manuais geram evento auditável com ator/data/resultado.
- [ ] Erro de banco não informa sucesso nem apaga formulário.
- [ ] Conectores não iniciam chamada na interface ou servidor.
- [ ] Restore de teste preserva histórico.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | Configuração, auditoria e connector gate inexistentes | Testes de integração de CRUD admin, activity e endpoint de integração | Falha nos critérios F1.4 | Saída de suíte focada |
| GREEN | Operar manualmente com cada papel e tentar iniciar Asana/Drive | Teste de mutações/API + inspecionar chamada de rede | Ações permitidas auditadas; conector bloqueado e sem rede | Logs redigidos e contagem de rede zero |
| REFACTOR/REGRESSÃO | Indisponibilidade de banco, opção desativada, backup/restore e retry | Exercitar cenários de falha/recuperação | Sem falsa confirmação, duplicata ou perda de histórico | Relatório de recuperação |

**Dados/fixtures:** Admin, CS, Engenharia e Legais; opções ativas/inativas; eventos antes/depois; falha simulada de banco.<br>
**Caminhos de erro obrigatórios:** papel insuficiente, gravação parcial, database offline, feature integration disabled e restore.<br>
**Evidência exigida:** auditoria redigida, chamada de rede inexistente e restore validado.

## Handoff e operação

- **Como demonstrar:** alterar lista e registro manual, consultar atividade, abrir integrações planejadas e confirmar estado inativo.
- **Como operar depois:** Admin cuida de usuários/listas; donos de área corrigem registro por nova versão/evento.
- **Como monitorar:** erros de gravação, mudança de configuração e backups.
- **Pendência conhecida:** conectores reais permanecem na Fase 2; retenção detalhada precisa de política Thórus.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| F1-08 | Auditoria manual e recuperação de erro | Engenharia | F1.4 | Ações críticas auditadas; falha não gera sucesso falso | Auditoria; Falhas e recuperação; TDD | Log e cenário de falha | F1.1,F1.2 | Planejada |
| F1-10 | Aceite manual e backup/restore | Thórus CS | F1.4 | Fluxo manual fecha e restore conserva relações/histórico | Aceite verificável; TDD regressão | Demonstração e log de restore | F1-04–F1-09,F1-11–F1-16 | Planejada |
| F1-14 | Configurações administrativas de usuários e listas | Engenharia | F1.4 | Admin altera config; valor antigo permanece legível | Administração; RN-F1.4-01 | Evidência antes/depois | F1-02,F1-04,P5/P8/P9 | Planejada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
