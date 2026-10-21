# Registro — F1-22: validação de filtros, privacidade e estados de retorno da carteira (SPEC-1-006-CA-02)

**Data:** 2026-10-21 · **Validação (dono Thorus):** Cristiano (owner, agindo como Thórus Admin/champion) · **Execução da prova:** Mara (assistente) · **Nenhuma linha de código foi alterada** (app permanece v0.0.12 `5e3355a`).

## Matriz query × perfil (14 cenários, todos aprovados)

| # | Cenário | Consulta | Resultado observado | Veredito |
|---|---|---|---|---|
| S01 | Padrão Admin | `/projetos` | A visível, B ausente, total "1 projeto", página 1 de 1 | ✓ |
| S02 | Arquivados Admin | `?archived=1` | B visível, A ausente (não arquivado), total "1 projeto" | ✓ |
| S03 | Busca por B na visão ativa | `?q=aurora` | "Nenhum resultado para estes filtros" — B não vaza | ✓ |
| S04 | Busca por B em arquivados | `?archived=1&q=aurora` | B visível (Admin, visão explícita) | ✓ |
| S05 | UF de B | `?archived=1&uf=PR` | B visível | ✓ |
| S06 | Filtro CS | `?cs=Carlos+CS` | A visível, B ausente | ✓ |
| S07 | Página inexistente | `?page=5` | Página presa em 1 de 1, A visível (clamp) | ✓ |
| S08 | Busca em maiúsculas | `?q=VALE` | A visível (normalização caixa/acento) | ✓ |
| S09 | Combinação AND | `?phase=Projeto executivo&uf=SC` | A visível, B ausente | ✓ |
| S10 | Cliente de B | `?archived=1&client=Incorporadora Vale Verde` | B visível (Admin) | ✓ |
| S11 | Paginação 50 | `?pageSize=50` | A visível, total correto | ✓ |
| S12 | Ordenação próximo prazo | `?archived=1&sort=next_due` | B primeiro (único registro da visão) | ✓ |
| S13 | Retorno do navegador | `?uf=SC` → Dashboard → voltar | URL `?uf=SC` restaurada, lista correta (CA-02 literal) | ✓ |
| S14 | Sentinela com troca real de perfil | seletor "ver como" → Atendimento/CS | Carlos CS na identidade; A visível; **B ausente em lista, total e opções** | ✓ |

## Critério fechado

- **SPEC-1-006-CA-02** — "Filtro autorizado e paginação continuam corretos ao voltar; projeto B nunca aparece em resultado/total": provado por S13 (retorno restaura URL/filtros/página) e S14 (sentinela com troca real de perfil, incluindo a lacuna de prova que a automação não cobria na F1-06). Resposta obsoleta: debounce 300 ms com timer mais recente vencendo o anterior, verificado na F1-06 e reexercitado em S03/S08.

## Observações

- F1-08 (SPEC-1-014) foi avaliada nesta rodada e considerada **prematura**: a rota `/projetos/{id}/atividade` depende do domínio da SPEC-1-008 (abas do projeto, não entregue) e nenhuma mutação com escopo de projeto existe ainda. Trava registrada em STATUS/changelog.
- Divergências encontradas: nenhuma. Não houve correção silenciosa — nada precisou de correção.
