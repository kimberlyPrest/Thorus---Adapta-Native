# Fase 1 — Tasks gerais

**Entrega:** MVP interno manual. Cada task produz incremento demonstrável de uma rota: interface + dados + autorização + persistência + recuperação/auditoria aplicáveis. Nenhuma task nesta tabela ativa integração de negócio. Ver [índice das SPECs verticais](specs/00-INDICE.md).

Os contratos compartilhados D1–D5 e C1–C5 definem padrão de interface/dados; tasks continuam vinculadas a uma SPEC de rota por vez. Status inicial `Planejada`.

| ID | Task | Dono | SPEC | Critério binário | Subseção SPEC | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| F1-01 | Confirmar e aprovar campos mínimos do cadastro manual | Thorus | SPEC-1-007 | Produto e Thórus aprovam o dicionário de campos C2 aplicável ao projeto piloto. | Decisões de campos e obrigatoriedade | SPEC-1-007-CA-01; cenário UI/API, releitura e estado persistido | Dicionário com campo, tipo, obrigatoriedade, permissão e exemplo fictício aprovado. | P1/P2 respondidas para dados do projeto; campo pessoal sem finalidade não entra. | Parar ao encontrar campo obrigatório sem finalidade, origem o