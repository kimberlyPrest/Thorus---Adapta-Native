# Registro F1-16 — Design system, estados base, teclado e responsividade

**Data:** 2026-10-29
**Task:** F1-16 · **SPEC:** SPEC-1-001 · **Critério:** CA-02
**App:** Gestão de projetos (Skip 56846) · **Versões:** v0.0.29 `4bf7a5f` → v0.0.30 `a2af39e`
**Champion:** Amanda (Thórus Admin) · **Canal:** web

## Autorização

Formulário em 2026-10-29, com duas respostas:
- Autorização para implementar: **"Autorizo implementar o plano"**
- Escopo da troca de classes: **"Todas as páginas entregues"**

O escopo explícito das páginas foi decisivo: o plano original previa shell + Dashboard, e o champion ampliou para todas as telas entregues.

## Baseline (RED, provado antes de escrever código)

| Item | Antes | Contrato D2 |
|---|---|---|
| Ação primária | `--primary: 221.2 83.2% 53.3%` = azul shadcn **#2563EB** | teal **#0F766E**, hover #115E59 |
| Anel de foco | herdado do azul | mesma cor da ação primária |
| Tipografia | **Inter** via `@import` do Google Fonts | `system-ui` |
| Cores por classe arbitrária | **266 hex em 17 cores** (39 só no AppShell) | zero hex literal |
| Atalho para o conteúdo | inexistente | skip-link + `main` focalizável |

## O que foi feito

1. **Tokens D2** em `src/main.css` e mapeamento em `tailwind.config.ts`: `--primary` 175 77% 26% (#0F766E), `--primary-hover` #115E59, `--ring` na cor primária, `--border` #CBD5E1, `--border-soft` #E2E8F0, `--muted-foreground` #475569, `--muted-soft` #94A3B8, `--destructive` #B91C1C (+ soft/border), `--success`, `--warning`, `--info` (+ soft/border).
2. **Tipografia** trocada para stack `system-ui`; import do Google Fonts removido (sem requisição de terceiro, sem flash de fonte).
3. **Troca de 266 hex → 0** em 8 arquivos: `AppShell`, `Dashboard`, `Projects`, `ProjectDetail`, `ProjectActivity`, `Profiles`, `Integrations` e **`Index` (tela de login)**.
4. **Teclado:** skip-link "Pular para o conteúdo" como primeiro alvo do Tab; `main#conteudo-principal` com `tabIndex={-1}` e `outline-none`; anéis de foco em menu, botões e campos.
5. **Bug de acesso corrigido** — ver abaixo.

## Bug de acesso (causa raiz do teste reprovado)

O primeiro teste humano voltou "Não funcionou — Não consegui acessar com login e senha passados". A investigação separou três camadas:

1. **Credencial:** `POST /api/collections/users/auth-with-password` com `admin@thorus.demo` devolveu token e o registro `hm7j5jcqgruj9mi` — senha íntegra.
2. **Host:** o preview devolvia **HTTP 522 / timeout** (Cloudflare) enquanto o PocketBase respondia `200` em 0,04 s — indisponibilidade real, mas transitória.
3. **Fluxo no navegador (a causa raiz):** ao autenticar, `src/pages/Index.tsx` executava `window.location.href = '/'`. Como `/` **é a própria rota da tela de login** em `App.tsx`, o usuário entrava, via o aviso "Sessão ativa" e permanecia preso na tela de entrada — sem nunca alcançar o shell. Corrigido para `/dashboard`.

**Prova de ponta a ponta após a correção:** logout → login `admin@thorus.demo` / `Thorus2026!` → URL final `/dashboard` com shell autenticado.

## Verificação automática

QA da plataforma verde nas duas versões (setup, estática, build, integrações). `test.ran: false` — o runner Vitest não pode ser instalado nesta fase (`package.json` é `preventAI`), dívida mantida e registrada no código.

## Teste humano

Aprovado pelo champion com **"Funcionou"** em 2026-10-29, no reteste após a correção de acesso.

## Dívidas

- Matriz **viewport/foco/contraste** é roteiro humano, não suíte automatizada (runner indisponível). Viewports 375/768/1440 e navegação por teclado foram exercitados no roteiro, não em teste automatizado.
- Contraste verificado por inspeção visual no roteiro; não houve cálculo de razão de contraste (o contrato D2 pede verificação, que ficou no portão humano).
- `src/pages/Index.tsx` mantém estilos próprios da tela de entrada (não usa o shell); agora consome os mesmos tokens, mas não foi refatorada para componentes D2.

## Aprendizados (registrados em `06_notas/aprendizado-continuo/controle.md`)

1. **Diagnóstico de "não consigo acessar":** separar infraestrutura de código antes de culpar credenciais — testar credencial na API, checar o host, e só então reproduzir o fluxo no navegador. Sem o terceiro passo, o bug de roteamento teria ficado escondido atrás da indisponibilidade do preview.
2. **Auditoria de tokens:** enumerar todas as rotas de `App.tsx`, incluindo as que ficam fora do shell; a tela de login ficou de fora da primeira passada e manteve azul residual. Critério de aceite numérico (266 → 0) em vez de inspeção visual.
