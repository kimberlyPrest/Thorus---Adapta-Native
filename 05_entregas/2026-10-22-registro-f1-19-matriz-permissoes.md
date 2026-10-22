# Registro F1-19 — Matriz de permissões em servidor (SPEC-1-004-CA-02)

**Data:** 2026-10-22 · **App Skip:** v0.0.13 `e8d372e` → v0.0.14 `f0edc9b` → v0.0.15 `6ae862f` · **QA:** verde nas três versões

## O que foi entregue

- **Migration 0004** (`pocketbase/migrations/0004_permissions_schema.js`): coleções `roles`, `role_permissions` (UNIQUE role+resource+action), `policy_state` (policy_version) e `activity_events` (append-only — create/update/delete `null` via API; escrita apenas server-side).
- **Migration 0005** (`0005_seed_permissions_matrix.js`): seed idempotente da matriz assinada na F1-02 — Admin 36, CS 13, Engenharia 10, Legais 6, Liderança 7 grants — e `policy_state` v1.
- **Rotas Admin-only** (`pocketbase/hooks/admin_roles_list.js`, `admin_permissions_preview.js`, `admin_permissions_save.js`): GET `/backend/v1/admin/roles`; POST `.../{key}/permissions/preview` (delta + afetados, sem gravar); PUT `.../{key}/permissions` (transação: substitui grants + incrementa policy_version + grava `activity_events` — primeira entrega C3). Validações: motivo 3–500, write exige read (C4), célula administrativa do Admin fixa, conflito de policy_version **409** (ApiError).
- **Página `/configuracoes/perfis`** (`src/pages/Profiles.tsx`, service `src/services/adminPermissions.ts`): gate pelo auth real (useAuth — não o seletor de demonstração), abas por perfil, matriz papel × ação × escopo, "Revisar e salvar" com motivo obrigatório, tratamento de 409, células administrativas fixas, "Restaurar base do piloto".

## Provas de API (verificação independente, reprovadas do zero)

| Prova | Resultado |
|---|---|
| Preview == delta salvo | PASSOU — revoked `[tasks:complete]` idêntico em preview e save |
| Salvar incrementa policy_version | PASSOU — 1→2 |
| Reenvio com versão obsoleta | PASSOU — recusado; **409 real no transporte** após correção (antes 400, divergência de contrato corrigida em revisão) |
| Write sem read | PASSOU — recusado (400, C4) |
| Motivo < 3 caracteres | PASSOU — recusado (400) |
| CS nas rotas admin | PASSOU — 403 |
| Anônimo | PASSOU — 403 |
| Evento de auditoria | PASSOU — `activity_events` gravado na mesma transação |
| Matriz assinada intacta | PASSOU — policy 4, CS com `tasks:complete` restaurado |

## Teste humano

Roteiro de 6 passos enviado em 2026-10-21; champion confirmou "Funcionou" (2026-10-22, canal web). Estado da máquina: `implementando → aguardando_teste_humano → concluida`.

## Dívidas

- `affected_users` = 1 até provisionamento por usuário (F1-14/F1-18).
- Suite Vitest sem runner (preventAI da plataforma) — dívida mantida.
- Dívida "role admin/cs" da F1-09 fechada por esta entrega (campo `role` de `users` agora governado pelas coleções de permissões).
