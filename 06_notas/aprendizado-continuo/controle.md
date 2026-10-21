# Controle de aprendizado contínuo

| Data | Origem | Aprendizado | Evidência |
|---|---|---|---|
| 2026-10-20 | F1-02 — validação da matriz pela Thórus | Regras de negócio confirmadas verbalmente podem divergir do contrato: ao detectar divergência material (ex.: coaprovação vs. `decided_by` único), registrar DÚVIDA no changelog e oferecer dois caminhos — emenda de SPEC ou revisão da regra pelo cliente — em vez de inferir. A revisão da regra pelo cliente (aprovador único designado, CS avisado) resolveu sem emenda, pois o contrato já modelava a regra. | changelog 2026-10-07 (DÚVIDA), 2026-10-16 (revisão P9), 2026-10-20 (validação F1-02); estado-atual.md |
| 2026-10-20 | F1-02 — fechamento | Pré-condições documentais (P8, B1) destravam a task antes do portão humano; manter o portão separado e reapresentar o roteiro de validação quando o usuário pedir execução. | 06_notas/2026-10-16-registro-b1.md; changelog 2026-10-16 |
| 2026-10-21 | F1-01 — dicionário de campos | Resposta de questionário dada em conversa (P1/P2 em 07/10) sem registro formal no repositório deixa a pré-condição da task indefinida e atrasa a seleção; registrar a decisão no repo no momento da conversa (nota em 06_notas ou changelog) faz parte do fechamento, não é opcional. | changelog 2026-10-21; 06_notas/2026-10-21-registro-f1-01-dicionario.md |
