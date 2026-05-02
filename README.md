# Dashboard Turma B — v2.1 (Melhorias Maio 2026)

## O que há de novo nesta versão

- **Painel HH Apontado × HH Programado** — novo card com barra de progresso e % de aderência
- **KPI atualizado** — meta de dias × 7h explicada nos cards
- **Tooltip do gráfico de barras** — exibe data completa DD/MM/AAAA
- **E-mail do rodapé** — substituído por endereço corporativo
- **Cálculo correto de HH Programado** — baseado em dias com lançamento × 7h

## Regras de negócio implementadas

| Indicador | Fórmula |
|-----------|---------|
| HH Programado | `dias_trabalhados × 7h` |
| HH Apontado | `horas_total (OPER + todos os tipos de manutenção)` |
| Aderência HH | `(HH Apontado / HH Programado) × 100%` |
| Horas totais nominais | `dias × 8h × nº operadores` |
| Horas úteis nominais | `dias × 7h × nº operadores` |

**Nota**: A regra "cada dia com hora lançada = 7h de manutenção planejada + 21h operacional" aplica-se ao nível de equipe. O painel HH × HH usa o indicador individual (por operador selecionado) ou somado (_ALL).

## Arquivos incluídos

```
index.html       ← Dashboard principal (único arquivo para GitHub Pages)
logo_OPE.png     ← Logo Operação
logo_MNT.png     ← Logo Manutenção
logo_report.png  ← Ícone Relatório AppSheet
README.md        ← Este arquivo
PASSO_A_PASSO.md ← Guia de deploy e atualização de dados
```

## Deploy no GitHub Pages

Ver `PASSO_A_PASSO.md` para instruções completas.
