# Architecture

## System Overview

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                            DATA SOURCES                                      │
│                                                                              │
│   ┌─────────────────────┐          ┌──────────────────────┐                  │
│   │   UAU-based ERP      │          │   SharePoint          │                  │
│   │   (SQL Server)       │          │   (Reference Data)    │                  │
│   │                      │          │                       │                  │
│   └─────────┬───────────┘          └──────────┬───────────┘                  │
│             │                                  │                              │
└─────────────┼──────────────────────────────────┼──────────────────────────────┘
              │                                  │
              ▼                                  ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                          POWER BI LAYER                                      │
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐    │
│   │                    BI CONTROLADORIA (MASTER)                         │    │
│   │                    594 DAX measures                                  │    │
│   │                                                                      │    │
│   │   Core Domains:                                                      │    │
│   │   - Banco Gerencial (bank balances)                                  │    │
│   │   - Processos de Pagamento (payment processes)                       │    │
│   │   - Fechamento Financeiro (financial closing)                        │    │
│   │   - Fechamento Contabil (accounting closing)                         │    │
│   │   - Juros e Multas (penalties & late fees)                           │    │
│   │   - Extrato Auditoria (bank reconciliation)                          │    │
│   │   - Emissao Pagamentos (payment emission)                            │    │
│   │   - Mapa (cash position map)                                         │    │
│   │   - Pagamentos Fora do Prazo (late payments)                         │    │
│   └──────────────────────────┬──────────────────────────────────────────┘    │
│                              │                                               │
│              ┌───────────────┼───────────────────────┐                       │
│              │               │                       │                       │
│              ▼               ▼                       ▼                       │
│   ┌─────────────────────────────────────────────────────────────────────┐    │
│   │                     9 VARIANT DASHBOARDS                             │    │
│   │                                                                      │    │
│   │   ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐    │    │
│   │   │ Bancos Gerenciais│ │ Conciliacao      │ │ Contas Pagas     │    │    │
│   │   └──────────────────┘ └──────────────────┘ └──────────────────┘    │    │
│   │   ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐    │    │
│   │   │ Fechamentos      │ │ Juros e Multa    │ │ Juros e Multa 1  │    │    │
│   │   └──────────────────┘ └──────────────────┘ └──────────────────┘    │    │
│   │   ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐    │    │
│   │   │ Suprimentos      │ │ Fech. Gerencial  │ │ (future variant) │    │    │
│   │   └──────────────────┘ └──────────────────┘ └──────────────────┘    │    │
│   └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│   ┌─────────────────┐ ┌─────────────────┐ ┌────────────────────────────┐    │
│   │ BI TESOURARIA    │ │ BI FLUXO CAIXA  │ │ BI GESTAO DE TAREFAS      │    │
│   │ 576 measures     │ │ 186 measures    │ │ 1 measure                  │    │
│   └─────────────────┘ └─────────────────┘ └────────────────────────────┘    │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

## Key DAX Patterns

### 1. Semi-Additive Bank Balance (LASTNONBLANK)

Bank balances are **semi-additive** -- they should not be summed across time. The `LASTNONBLANK` pattern returns the balance as of the last date with data within the current filter context.

```dax
001 - SaldoGerencial =
CALCULATE(
    SUM('Saldos Gerenciais'[Saldo_sdcc]),
    LASTNONBLANK(
        VALUES('Saldos Gerenciais'[Data_sdcc]),
        DISTINCTCOUNT('Saldos Gerenciais'[Conta_sdcc])
    )
)
```

This pattern is used in both BI Controladoria and BI Fluxo Caixa (as `Saldo Atual`), ensuring that when a user filters by month, they see the closing balance rather than a sum of daily balances.

The Fluxo Caixa variant adds a `Saldo Bancario` wrapper that uses `ALL('Date')` to find the last non-blank date regardless of slicer selection:

```dax
Saldo Bancario =
VAR _lastdate =
    CALCULATE(
        MAX('Date'[Date]),
        FILTER(
            ALL('Date'),
            'Date'[Date] <= MAX('Date'[Date])
                && NOT(ISBLANK([Saldo Atual]))
        )
    )
RETURN
    CALCULATE([Saldo Atual], FILTER(ALL('Date'), 'Date'[Date] = _lastdate))
```

### 2. Cash Flow Model (Entry/Exit/Transfer Reconciliation)

The BI Fluxo Caixa model decomposes cash movement into four streams:

```
Opening Balance (Saldo PreviousMonth)
  + Fluxo Entradas (deposits/receivables)
  - Fluxo Saida (payments/disbursements)
  + Entrada Transferencias (incoming transfers)
  - Transferencia Saida (outgoing transfers)
  = Closing Balance (Entrada - Saida)
```

The `Entrada - Saida` measure ties all components together:

```dax
Entrada - Saida =
'M_SALDO BANCARIO'[Saldo PreviousMonth]
    + 'M_FLUXO'[Fluxo Entradas]
    - 'M_FLUXO'[Fluxo Saida]
    + 'M_FLUXO'[Entrada Transferencias]
    - 'M_FLUXO'[Transferencia Saida]
```

### 3. Quarter-Over-Quarter Variance (DATEADD)

Revenue and expense comparisons use `DATEADD` with quarterly offsets:

```dax
-- Current quarter vs. same quarter last year
Faturamento Trimestre Ano Passado =
CALCULATE([Fluxo Entradas], DATEADD('Date'[Date], -4, QUARTER))

-- Current quarter vs. previous quarter
Faturamento Trimestre Anterior =
CALCULATE([Fluxo Entradas], DATEADD('Date'[Date], -1, QUARTER))

-- Variance calculation
Var Faturamento Tri Tri 4 =
VAR Entrada = CALCULATE(M_FLUXO[Fluxo Entradas], DATEADD('Date'[Date], 0, QUARTER))
VAR Entrada_Tri_Anterior = CALCULATE([Fluxo Entradas], DATEADD('Date'[Date], -4, QUARTER))
RETURN
    IF(DIVIDE(Entrada, Entrada_Tri_Anterior) = 0, BLANK(),
       DIVIDE(Entrada, Entrada_Tri_Anterior) - 1)
```

The `-4, QUARTER` offset compares to the same quarter one year ago (4 quarters back), while `-1, QUARTER` compares to the immediately preceding quarter.

### 4. Working Capital Need (Necessidade de Caixa)

A simple but critical formula that combines current balance with payable obligations:

```dax
Necessidade de Caixa =
[Contas a Pagar] + 'M_SALDO BANCARIO'[Saldo Atual] + [Contas em Atraso]
```

Where:
- `Contas a Pagar` = future payables (DtPgto_Des > TODAY())
- `Contas em Atraso` = overdue payables (DtPgto_Des <= TODAY())
- Both are negated (shown as negative values) to represent cash outflows

### 5. Default Rate (Taxa Inadimplencia)

A slicer-driven measure using `SELECTEDVALUE` to let users input an assumed default rate:

```dax
Taxa Inadimplencia Valor = SELECTEDVALUE('Taxa Inadimplencia'[Taxa Inadimplencia])
```

This parameter table pattern allows end users to adjust projections by selecting a default rate percentage from a slicer, which then feeds into cash flow projections.

### 6. Financial Closing Status Tracking

The Controladoria model tracks both financial and accounting closing status across all SPEs:

```dax
-- Count of entities with OPEN financial closing
001 - AbertoFechFin =
IF(
    CALCULATE(COUNT('Status Fechamento Financeiro'[TIPO]),
        FILTER(..., [TIPO] = "ABERTO")) = BLANK(),
    0,
    CALCULATE(COUNT(...), FILTER(..., [TIPO] = "ABERTO"))
)

-- Average days to close
013 - Media de Dias Fechamento =
CALCULATE(
    AVERAGE('Fechamento Financeiro'[DiasFechamento]),
    FILTER('Fechamento Financeiro', 'Fechamento Financeiro'[CONT] = 1)
)
```

This enables a dashboard view showing which SPEs are closed, which are open, which are late, and the average closing time -- critical for BPO SLA compliance.

### 7. Template-Variant Pattern

The master Controladoria model contains all 594 measures organized by `DisplayFolder`. Each variant dashboard:

1. Starts from a copy of the master `.pbix`
2. Keeps only the pages relevant to its domain
3. May add a small number of variant-specific visuals
4. Connects to the same SQL Server data source

See [template-variants.md](template-variants.md) for the complete breakdown of how this pattern works in practice.
