# Registro — F1-06: carteira com busca, filtros e paginação autorizados (SPEC-1-006)

**Data:** 2026-10-21 · **Autorização e teste humano:** Cristiano (owner, agindo como Thórus Admin/champion) · **Implementação:** Mara (assistente)

## Escopo entregue

Rota `/projetos` conforme a tabela "Página e comportamento detalhado" da SPEC-1-006, sobre dados C5 locais (projetos A e B; CS atribuído somente a A; B arquivado como sentinela de vazamento). Autorização local com semântica C4: o universo autorizado é filtrado ANTES de busca, filtros, contagem e paginação — nenhum resultado oculto entra no total. Sem API real: o binding C3 (GET /api/projects) e as coleções chegam com a F1-15; dívida marcada no código (`adapta-divida`).

## Arquivos (app Skip, projeto 56846)

- `src/lib/fixtures.ts` — identidade e capabilities de demonstração (Admin/CS, recorte da matriz C4 aprovada na F1-02) + fixtures C5 (projetos A/B, memberships).
- `src/lib/portfolio.ts` — consulta local: autorização antes de tudo; busca normalizada (sem acento/caixa, trim, ≤120); filtros AND (cliente, fase, situação, CS, responsável técnico, UF, arquivados); ordenação nome/atualizado/próximo prazo (sem prazo ao final, empate por id); paginação 25/50 com clamp de página.
- `src/lib/demoRoleStore.ts` + `src/context/DemoRoleContext.tsx` — perfil "ver como" em store singleton compartilhado entre shell e páginas (sobrevive a remontagens do StrictMode).
- `src/components/shell/AppShell.tsx` — consome o perfil compartilhado (menu e identidade).
- `src/pages/Projects.tsx` — header com contador autorizado; busca com debounce 300 ms, Enter imediato e indicador "Buscando…"; 6 filtros; Ver arquivados; Limpar filtros; ordenação; tabela D2 (cards no mobile); paginação; filtros/página/ordenação na URL (D5/D1).
- `src/App.tsx` — DemoRoleProvider adicionado.

Versões do app: 0.0.5 (`91b5d57`), 0.0.6 (`2208d5f` — store singleton), 0.0.7 (`7f71374` — filtros CS/Resp. técnico). QA pipeline verde em todas (setup/staticAnalysis/build/integrações; test ran:false — runner ausente, dívida já marcada).

## Verificação automática (preview, por automação)

- Admin: "1 projeto" ativo (A); `?archived=1` exibe A+B ("2 projetos"); "Limpar filtros" restaura a visão ativa.
- Busca `?q=vale` filtra na URL e mantém A; B nunca aparece em lista, total ou filtro.
- Filtro CS = "Carlos CS": URL `?cs=Carlos+CS`, A visível, B ausente.
- CA-02 parcial por automação: URL preserva filtros; voltar/avançar e resposta obsoleta dependem de interação humana (roteiro enviado).

## Dívidas marcadas

- Dados e autorização locais (fixtures C5) até a migration da F1-15 e o binding C3 real.
- Suite Vitest não executável (package.json preventAI na plataforma) — QA pipeline + roteiro manual.
- Tokens D2 por classe arbitrária (pacote de design ainda não aprovado).
- Ações por linha (Abrir/Editar/Arquivar) e CTA Novo projeto desabilitados — SPEC-1-008/F1-15.

## Critério fechado

- **SPEC-1-006-CA-01** — "Consulta, total, filtros e próxima data representam apenas os projetos autorizados": verificado no preview (sentinela B ausente para CS em lista/total/filtro; contadores e opções de filtro derivados só do universo autorizado) + teste humano aprovado pelo champion ("funcionou", 2026-10-21 11:51, canal web).
- CA-02/CA-03: parcialmente exercidos (URL, vazio, sem resultado, carregando, erro e acesso negado implementados); validação formal é da F1-22 (dono Thorus).
