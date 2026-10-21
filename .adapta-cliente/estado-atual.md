# Estado atual — Adapta Cliente

- task_id: F1-03
- champion: Amanda (Thórus Admin)
- spec: 04_fase-atual/specs/spec-07-cadastro-projeto.md
- etapa: concluida
- autorizacao_implementacao: nao_aplicavel — F1-03 é documento de política técnica (modelo relacional), sem implementação; a migration é da F1-15
- teste_humano: aprovado + 2026-10-21T10:37 (canal web) — respostas do formulário: "Aprovo o modelo"; versionamento C2 confirmado; FKs RESTRICT; unicidade de code e normalized_name; índices confirmados; provisionamento por migration versionada (padrão B1)
- verificacao_automatica: passou — critério SPEC-1-007-CA-02 fechado: modelo relacional e constraint list de clients/projects versionados em 06_notas/2026-10-21-registro-f1-03-modelo-relacional.md; DÚVIDA registrada no changelog (unicidade de normalized_name vs. permissão de nomes iguais em SPEC-1-007) para decisão do consultor antes da F1-15
- aprendizado: capturado:06_notas/aprendizado-continuo/controle.md
- ultima_acao: F1-03 concluída em 2026-10-21 — modelo relacional aprovado pela Thórus e registrado; fase.md, STATUS.md, changelog.md e estado atualizados
- proxima_acao: aguardar pedido do champion para selecionar a próxima task (F1-05 é a próxima candidata de implementação; F1-15 depende da DÚVIDA do normalized_name; F1-19 depende de runtime provisionado; F1-04/F1-14 dependem de B3)
- atualizado_em: 2026-10-21T10:38:47-03:00
