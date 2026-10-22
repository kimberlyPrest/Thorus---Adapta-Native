# Registro F1-24 — Equipe e aprovadores do projeto (SPEC-1-008-CA-01)

**Data:** 2026-10-22 · **App Skip:** v0.0.16 `3b80f6f` → v0.0.20 `05f156a` · **QA:** verde nas cinco versões

## O que foi entregue

- **Migration 0006** (`pocketbase/migrations/0006_project_team.js`): coleções `project_memberships` (UNIQUE projeto+usuário; remoção = `active=false`, autoria preservada), `project_approvers` (UNIQUE projeto+disciplina — aprovador único, regra P9) e `project_state` (version por projeto; dívida até a F1-15), com seed espelhando as fixtures C5.
- **Rotas server-side** (`pocketbase/hooks/project_detail_get.js`, `project_team_put.js`): GET `/backend/v1/projects/{id}` (resumo + equipe + aprovadores + elegíveis; Admin lê tudo, não-membro 404 genérico, anônimo 401); PUT `.../team` transacional (memberships + designações + bump de version + `activity_events` — C3). Validações: aprovador precisa ser membro ativo com `definitions.decide` no papel; 409 em expectedVersion obsoleta; 403 não-Admin.
- **Página `/projetos/{id}`** (`src/pages/ProjectDetail.tsx`, service `src/services/projectDetail.ts`): header, resumo, equipe com badge de aprovador, drawer "Gerenciar equipe" (membros + aprovador por disciplina, só membros ativos), 409 tratado, estados D3, zero egress; links na carteira (`src/pages/Projects.tsx`).

## Provas de API (verificação independente, reprovadas do zero — 9 cenários)

| Prova | Resultado |
|---|---|
| GET Admin | PASSOU — equipe 3 ativos, aprovador Ana Admin/estruturas, version 5 |
| GET CS membro (proj-a) | PASSOU — 200 |
| GET CS sem vínculo (proj-b) | PASSOU — 404 genérico |
| GET anônimo | PASSOU — 401 |
| PUT com expectedVersion obsoleta | PASSOU — 409 |
| PUT como CS | PASSOU — 403 |
| Aprovador sem definitions.decide | PASSOU — 400 |
| Membro removido perde acesso | PASSOU — 404 na requisição seguinte |
| Membro readicionado recupera | PASSOU — 200 |
| Auditoria | PASSOU — `projects update team:proj-a` gravado na transação |

## Fluxo de UI (preview)

Drawer aberto → aprovador designado por disciplina → "Salvar equipe" → "Equipe salva — versão 5" → badge "Aprovador · estruturas" visível → persistência confirmada por releitura de API. Projeto B (Admin, membro) exibido corretamente; negação CS→proj-b provada por API.

## Teste humano

Roteiro de 6 passos enviado em 2026-10-22; champion confirmou "Funcionou" (2026-10-22, canal web). Estado da máquina: `implementando → aguardando_teste_humano → concluida`.

## Dívidas

- Dados de projeto espelhados em constante server-side e `version` em `project_state` até a F1-15 criar a coleção `projects`.
- Gate de gestão de equipe por role admin até `projects.manage_team` ser exposto pela matriz (F1-14/F1-18).
- Abas técnicas e arquivamento nas SPECs próprias (F1-36/F1-37).
- Suite Vitest sem runner (preventAI) — dívida mantida.

## Efeito na fila

F1-08 (atividade/auditoria, SPEC-1-014) volta a ser elegível: primeira mutação com escopo de projeto entregue (PUT team transacional com auditoria).
