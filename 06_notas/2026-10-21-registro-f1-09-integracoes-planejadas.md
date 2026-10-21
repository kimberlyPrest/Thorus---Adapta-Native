# Registro — F1-09: mapeamentos e GIDs externos sem integração ativa (SPEC-1-016)

**Data:** 2026-10-21 · **Autorização e teste humano:** Cristiano (owner, agindo como Thórus Admin/champion) · **Implementação:** Mara (assistente)

## Escopo entregue

Rota `/configuracoes/integracoes` conforme a SPEC-1-016: Admin cadastra mapeamentos Asana/Drive (campo externo → campo interno de allowlist C2, transform identity/text/date/enum_mapping, draft/reviewed) e vínculos externos de projeto (workspace/GID/tipo/permalink opcionais), tudo como rascunho persistido no PocketBase. `enabled=false` garantido no servidor (hook model); nenhuma chamada externa; nenhum botão Conectar/Sincronizar/Testar/Upload; nenhum campo de token/segredo. P1/P2/P3 não respondidos não bloqueiam (a SPEC permite rascunho com GIDs fictícios).

## Arquivos (app Skip, projeto 56846)

- `pocketbase/migrations/0001_integration_mappings.js` — coleções `integration_mappings` e `integration_links` (regras Admin-only C4, versionamento C2, índice UNIQUE de dedupe e índice parcial de GID).
- `pocketbase/migrations/0002_users_role_field.js` — campo `role` (select admin/cs) na coleção auth `users`, exigido pelas regras C4 (`@request.auth.role`). Dívida: matriz completa de perfis na F1-19.
- `pocketbase/migrations/0003_fix_enabled_required.js` — remove `required` do BoolField `enabled` (o PocketBase rejeita `false` como vazio); o hook continua forçando `false` no servidor.
- `pocketbase/hooks/integration_mappings_validations.js` — model hooks com validações inline (JSVM não expõe top-level aos callbacks): enabled forçado a false na criação, enabled=true recusado no update, allowlist de campo interno, enum_map obrigatório para enum_mapping, revision C2 (create=1, update=original+1), GID duplicado recusado, textos externos ≤120.
- `src/services/integrations.ts` — service layer (única que fala com o PocketBase); mapeia 400+campo revision → 409.
- `src/pages/Integrations.tsx` — página Admin-only com banner "Fase 2", tabelas de mapeamentos/vínculos, diálogos de criação, enum_map com pares, estados D3 (carregando/vazio/erro com Tentar novamente/acesso negado).
- `src/App.tsx` + `src/components/shell/AppShell.tsx` — rota `/configuracoes/integracoes` e item do menu Admin habilitado.

Versões do app: 0.0.9 (`4a80144`), 0.0.10 (`4ac4cc5`), 0.0.11 (`29febe6`), 0.0.12 (`5e3355a`). QA pipeline verde em todas (test ran:false — runner ausente, dívida mantida).

## Verificação automática (provas de API e preview)

- CA-01: mapeamento criado (enabled=false forçado pelo hook, revision=1); update para reviewed/revision 2 ok; vínculos criados com e sem GID; dados relidos após recriação.
- CA-02: enabled=true recusado; campo interno fora da allowlist recusado; GID duplicado recusado; revision obsoleta recusada ("deve ser 3"); update sem revision recusado.
- CA-03: zero botão Conectar/Sincronizar/Testar/Upload na página; zero input de token/segredo; nenhuma chamada externa (só coleções locais); listagem anônima retorna 0 itens (regras C4 filtram, sem vazamento).
- Revalidação independente (pós-teste humano): enabled=true recusado, GID duplicado recusado, sem-token lista vazia, dados persistem — todos reprovados/reprovados conforme esperado.

## Dívidas marcadas

- `project_id` do vínculo como texto (fixtures C5) até a F1-15 criar a coleção `projects`.
- Campo `role` em `users` com recorte admin/cs — matriz completa de perfis na F1-19.
- Suite Vitest não executável (package.json preventAI na plataforma) — QA pipeline + roteiro manual.
- activity_events (SPEC-1-014/F1-08) ainda não existe — auditoria desta rota fica para a F1-08.

## Critério fechado

- **SPEC-1-016-CA-01** — "Mapeamento/vínculo persistem com revisão/IDs opcionais sem bloquear uso manual": provas de API (criação/leitura/update com revision) + teste humano aprovado pelo champion ("Funcionou", 2026-10-21 12:29, canal web).
- CA-02/CA-03 implementados e provados na mesma entrega; validação formal da F1-33 (dono Thorus) confirma os cenários de rejeição depois.
