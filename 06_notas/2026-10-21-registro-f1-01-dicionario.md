# Registro F1-01 — Dicionário de campos do cadastro manual

**Data:** 2026-10-21 · **Task:** F1-01 · **SPEC:** SPEC-1-007 (cadastro de projeto/cliente) · **Critério fechado:** SPEC-1-007-CA-01 · **Aprovação:** Thórus Admin (Cristiano Schneider, canal web, 2026-10-21 09:40 — "Aprovo o dicionário")

## Decisão

O dicionário-base é o contrato C2 do plano, sem alterações: o mínimo obrigatório para criar um projeto permanece **nome + cliente**. Nenhum campo adicional foi tornado obrigatório e nenhum dado pessoal (CPF/CNPJ/endereço/contato pessoal) é obrigatório — campo pessoal sem finalidade, origem e acesso definidos não entra (ponto de parada da task).

## Dicionário aprovado (campo, tipo, obrigatoriedade, permissão, exemplo fictício)

| Entidade | Campo | Tipo | Obrigatório | Permissão de escrita | Exemplo fictício |
|---|---|---|---|---|---|
| clients | name | texto 2–160 | Sim | projects.create (criação no drawer do cadastro) | Construtora Horizonte Ltda |
| clients | normalized_name | derivado (busca) | Automático | servidor | construtora horizonte ltda |
| projects | name | texto 2–160 | Sim | projects.create / projects.update | Residencial Vale Verde |
| projects | client_id | FK clients | Sim | projects.create / projects.update | (vínculo ao cliente acima) |
| projects | code | texto ≤30, único, maiúscula/trim | Não | projects.create / projects.update | TH-2026-014 |
| projects | city | texto ≤120 | Não | projects.create / projects.update | Blumenau |
| projects | state_code | enum UF | Não | projects.create / projects.update | SC |
| projects | contracted_scope | texto ≤4000 | Não | projects.create / projects.update | Projeto executivo de estrutura e fundações |
| projects | phase_option_id | FK catalog_options (project_phase) | Não | projects.create / projects.update | Projeto executivo |
| projects | status_option_id | FK catalog_options (project_status) | Não | projects.create / projects.update | Em andamento |
| projects | start_date | DATE | Não | projects.create / projects.update | 2026-11-02 |
| projects | target_date | DATE (≥ start_date) | Não | projects.create / projects.update | 2027-03-15 |
| projects | cs_user_id | FK users (ativo) | Não | projects.create / projects.update | (usuário CS fictício) |
| projects | technical_user_id | FK users (ativo) | Não | projects.create / projects.update | (usuário Engenharia fictício) |
| projects | notes | texto ≤4000 | Não | projects.create / projects.update | Cliente prefere reuniões quinzenais |

Regras confirmadas junto ao dicionário (SPEC-1-007): criador com escopo assigned vira membro automaticamente; cliente novo pode ter nome igual a existente (sugestão por normalização, sem deduplicação destrutiva); projeto e vínculos gravados atomicamente; mudança futura de obrigatoriedade exige emenda de produto, não toggle irrestrito.

## Pendências P1/P2 (registradas formalmente)

- **P1 (projetos/campos do piloto):** ainda não definido. Bloqueia apenas o mapeamento do piloto/integração (Fase 2); não bloqueia a Fase 1 manual.
- **P2 (fonte de verdade por campo):** ainda não definido. Decide a fonte de verdade da Fase 2; não bloqueia a Fase 1 manual. Operação manual sem conexão ao Asana confirmada; cópia de dados importados: manter até decisão posterior.

## Efeito

Nada foi implementado: task de validação de política, sem código ou migration. F1-15 (formulários) e F1-03 (modelo relacional) passam a usar este dicionário como teto — o que não está nele não pode ser exigido sem emenda de produto.
