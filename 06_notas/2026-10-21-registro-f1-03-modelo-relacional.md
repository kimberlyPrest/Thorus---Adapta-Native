# Registro F1-03 — Modelo relacional de clients/projects (aprovado para migração)

**Data:** 2026-10-21 · **Task:** F1-03 · **SPEC:** SPEC-1-007 (cadastro de projeto/cliente) · **Critério fechado:** SPEC-1-007-CA-02 · **Aprovação:** Thórus Admin (Cristiano Schneider, canal web, 2026-10-21 10:37 — "Aprovo o modelo")

## Decisões aprovadas

1. **Versionamento C2 confirmado:** `version` inteiro inicial 1 em clients e projects, com `created_at`, `created_by` e `updated_at`; edição exige versão atual (conflito → 409, sem sobrescrita).
2. **FKs de histórico com RESTRICT, sem cascade:** client_id, phase_option_id, status_option_id, cs_user_id e technical_user_id nunca apagam decisões/histórico em cascata.
3. **Unicidade:** `projects.code` único quando preenchido (trim/maiúscula, ≤30); `clients.normalized_name` único; `projects.name` sem unicidade (projetos podem ter o mesmo título).
4. **Índices de consulta confirmados:** projects por client_id, phase_option_id, status_option_id, archived_at e busca por normalized_name (clients).
5. **Provisionamento por migration versionada (padrão B1):** coleções criadas em `pocketbase/migrations/`; o app Skip não é alterado fora de task autorizada. A migration em si é da F1-15.

## Modelo (entidades desta task — contrato C2)

- **clients:** id, name 2–160 (obrigatório), normalized_name (único), version, created_at/by, updated_at. Sem CPF/CNPJ/endereço/contato pessoal obrigatório (dicionário F1-01).
- **projects:** id, code opcional único ≤30, name 2–160 (obrigatório), client_id FK RESTRICT (obrigatório), city ≤120, state_code enum UF, contracted_scope ≤4000, phase_option_id/status_option_id FK opcionais (RESTRICT), start_date/target_date DATE opcionais com target_date ≥ start_date, cs_user_id/technical_user_id FK opcionais (RESTRICT), notes ≤4000, archived_at, version, created_at/by, updated_at.

## DÚVIDA (para o consultor — não bloqueia o registro)

A unicidade de `clients.normalized_name` aprovada no formulário diverge da SPEC-1-007, que permite cadastrar cliente novo com nome igual a existente (sugestão por normalização, sem deduplicação destrutiva). O consultor decide antes da migration da F1-15: manter UNIQUE (emenda de SPEC) ou índice sem unicidade (leitura literal da SPEC). Enquanto não resolvida, a F1-15 não implementa essa constraint.

## Efeito

Nada foi implementado: documento de política técnica, sem migration ou código. Baseline do repositório executável verificado no Skip (projeto 56846, v0.0.3 `fe5cc94`, não publicado; somente coleção auth `users`; nenhuma migration; alteração pendente em `.skip.config.json` permanece intocada).
