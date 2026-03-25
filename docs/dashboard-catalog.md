# Dashboard Catalog

## 1. BI Controladoria (Financial Controlling, Master)

**Purpose**: Comprehensive financial controlling dashboard serving as the master template for all variant dashboards. Covers bank balance monitoring, payment process tracking, financial and accounting closing status, bank reconciliation, penalty analysis, payment emission workflow, and cash position mapping.

**Measures**: 594 DAX measures

**Complexity**: High

**Data Sources**:
- UAU-based ERP (SQL Server) -- primary transactional data
- SharePoint -- reference data

**Key DAX Measures**:

| Measure | Folder | Description |
|---------|--------|-------------|
| `001 - SaldoGerencial` | Banco Gerencial | Semi-additive bank balance using `LASTNONBLANK`. Returns the account balance as of the last date with data in the current filter context, preventing incorrect summation across time periods. |
| `001 - AbertoFechFin` | Fechamento Financeiro | Count of SPEs with financial closing status "ABERTO" (open). Uses `FILTER` on `Status Fechamento Financeiro` table. Returns 0 instead of BLANK for consistent card display. |
| `001 - PagamJurosMultas` | Juros e Multas | Total penalty and late fee payments, excluding acquisition costs and monetary adjustments (Atualizacao Monetaria, Reajuste de Valor). Filters out non-penalty line items. |
| `001 - StatusConciliacao` | Extrato Auditoria | Bank reconciliation status -- compares `SaldoFinalUAU` against `ValorFinalExtrato`. Returns "CONCILIADO" if they match, "NAO CONCILIADO" otherwise. |
| `001 - ProcessosEmAtraso` | Emissao Pagamentos | Count of payment processes with status "ATRASADO" (overdue). Part of the payment emission tracking workflow. |

**Special Techniques**:
- `LASTNONBLANK` for semi-additive bank balances
- `TOTALMTD` with `PREVIOUSMONTH` for month-over-month comparisons
- `FILTER` + `MAX(DataAlteracao)` pattern to get the latest status per entity
- Dual closing tracking (financial + accounting) with SLA day calculations
- `SELECTEDVALUE`-style slicer parameters for dynamic filtering

---

## 2. BI Tesouraria (Treasury)

**Purpose**: Treasury management dashboard focused on the same financial controlling domains as Controladoria (Financial Controlling) but with an emphasis on the payment emission pipeline -- tracking which invoices are in emission, which are overdue, which have been sent to the bank, and supplier-level aggregations.

**Measures**: 576 DAX measures

**Complexity**: High

**Data Sources**:
- UAU-based ERP (SQL Server)
- SharePoint

**Key DAX Measures**:

| Measure | Folder | Description |
|---------|--------|-------------|
| `001 - SaldoGerencial` | Banco Gerencial | Identical to Controladoria (Financial Controlling) -- semi-additive bank balance via `LASTNONBLANK`. |
| `009 - EnviadosABanco` | Emissao Pagamentos | Count of payment processes with `EnvioBanco = "ENVIADO AO BANCO"`. Critical for treasury teams tracking bank submission status. |
| `006 - SomaProcessosAtraso` | Emissao Pagamentos | Total monetary value of overdue payment processes. Filters by `StatusPagamento = "ATRASADO"`. |
| `001 - PagamJurosMultas` | Juros e Multas | Same penalty tracking as Controladoria -- sum of penalty values excluding acquisition costs and monetary adjustments. |
| `012 - StatusFechamento` | Fechamento Financeiro | Financial closing KPI -- retrieves the latest closing status using `MAX(DataAlt_hff)` filter. |

**Special Techniques**:
- Mirrors the Controladoria measure library almost entirely (576 vs. 594 measures)
- The 18-measure difference is due to the Controladoria master containing a few additional Contas a Pagar/Recebidas measures and test measures not carried over to Tesouraria
- Payment emission workflow tracking (A VENCER / ATRASADO / ENVIADO AO BANCO / NAO ENVIADO AO BANCO)
- Supplier distinct count (`DISTINCTCOUNT(EmissaoPagamentos[NomeFornecedor])`)

---

## 3. BI Fluxo Caixa (Cash Flow)

**Purpose**: Dedicated cash flow dashboard providing a complete view of money movement -- bank balances, entries (receivables/deposits), exits (payments/disbursements), inter-account transfers, quarterly variance analysis, and working capital need calculation.

**Measures**: 186 DAX measures

**Complexity**: Medium

**Data Sources**:
- UAU-based ERP (SQL Server) -- fact tables: FTO_ENTRADA, FTO_SAIDA, FTO_SALDOBANCARIO, FTO_TRANSFERENCIA, FTO_CONTAS_A_PAGAR, FTO_CONTAS_A_RECEBER
- DIM tables: DIM_Bancos, DIM_OBRAS, DIM_EAP, Date

**Key DAX Measures**:

| Measure | Table | Description |
|---------|-------|-------------|
| `Saldo Bancario` | M_SALDO BANCARIO | Sophisticated semi-additive balance using a two-step approach: first finds the `_lastdate` with non-blank data using `ALL('Date')`, then calculates `Saldo Atual` at that date. Handles gaps in daily balance data. |
| `Entrada - Saida` | M_FLUXO | Master cash flow reconciliation: `Saldo PreviousMonth + Fluxo Entradas - Fluxo Saida + Entrada Transferencias - Transferencia Saida`. Ties all cash flow streams together. |
| `Necessidade de Caixa` | M_FLUXO | Working capital need: `Contas a Pagar + Saldo Atual + Contas em Atraso`. Shows how much cash is needed to cover current and overdue obligations. |
| `Faturamento Trimestre Ano Passado` | M_FLUXO | Same-quarter-last-year revenue using `DATEADD('Date'[Date], -4, QUARTER)`. Enables year-over-year quarterly comparisons. |
| `Taxa Inadimplencia Valor` | Taxa Inadimplencia | Slicer-driven default rate parameter using `SELECTEDVALUE`. Lets users select an assumed default percentage that feeds into projections. |

**Special Techniques**:
- `LASTNONBLANK` with `DISTINCTCOUNT(EmpBcoConta)` for bank balance aggregation across multiple accounts
- `DATEADD` with `-4, QUARTER` for year-ago quarter comparison and `-1, QUARTER` for sequential quarter comparison
- Variance percentage calculation with `DIVIDE` and `BLANK()` fallback for zero denominators
- Separate entry/exit/transfer fact tables for clean cash flow decomposition
- `SUMX('DIM_Bancos', [Saldo Atual])` for cross-bank aggregation
- Net operating revenue calculation (`Receita Operacional Liquida`) combining gross revenue, deductions, and operating expenses via DIM_EAP cost center filtering

---

## 4. BI Gestao de Tarefas

**Purpose**: Lightweight task and project management tracker. Provides a simple count of task rows for monitoring workload and project status.

**Measures**: 1 DAX measure

**Complexity**: Low

**Data Sources**:
- Task data (likely SharePoint list or SQL Server table)

**Key DAX Measures**:

| Measure | Table | Description |
|---------|-------|-------------|
| `Contagem` | fTarefas | `COUNTROWS(fTarefas)` -- simple row count of the tasks fact table. Used as the primary KPI for task volume tracking. |

**Special Techniques**:
- Minimal DAX footprint -- the value of this dashboard is in the Power Query data model and visual layout rather than complex calculations
- The `fTarefas` naming convention (f-prefix for fact table) follows dimensional modeling best practices

---

## Cross-Dashboard Comparison

| Feature | Controladoria (Financial Controlling) | Tesouraria (Treasury) | Fluxo Caixa (Cash Flow) | Gestao Tarefas |
|---------|:------------:|:----------:|:-----------:|:--------------:|
| Bank balance (LASTNONBLANK) | Yes | Yes | Yes | -- |
| Payment process tracking | Yes | Yes | -- | -- |
| Financial closing status | Yes | Yes | -- | -- |
| Accounting closing status | Yes | Yes | -- | -- |
| Bank reconciliation | Yes | Yes | -- | -- |
| Penalty tracking | Yes | Yes | -- | -- |
| Payment emission workflow | Yes | Yes | -- | -- |
| Cash flow entry/exit | -- | -- | Yes | -- |
| Quarterly variance | -- | -- | Yes | -- |
| Working capital need | -- | -- | Yes | -- |
| Default rate parameter | -- | -- | Yes | -- |
| Task management | -- | -- | -- | Yes |
| Template-variant pattern | Yes (master) | No | No | No |
