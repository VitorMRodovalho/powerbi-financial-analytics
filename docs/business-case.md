# Business Case

## Industry Context

**Sector**: Brazilian real estate development and BPO (Business Process Outsourcing) financial operations.

In Brazil's real estate development market, companies typically create a separate SPE (Sociedade de Proposito Especifico -- Special Purpose Entity) for each development project. A single holding company or BPO provider may manage dozens of SPEs simultaneously, each requiring its own set of financial dashboards for controlling, treasury, cash flow, and compliance.

## The Problem

Financial controllers managing dozens of SPEs need standardized dashboards that can be deployed rapidly per entity without rebuilding from scratch. Each SPE requires:

- Bank balance monitoring (semi-additive, point-in-time)
- Payment process tracking (Cotacao, Processo de Pagamento, Compra Rapida, etc.)
- Financial and accounting closing status with SLA tracking
- Bank reconciliation (Conciliacao)
- Penalty and late fee monitoring (Juros e Multa)
- Cash flow forecasting with quarterly variance analysis
- Supplier management and procurement visibility

Building each dashboard individually is unsustainable. Maintenance becomes a nightmare when a formula change must be propagated across 30+ reports.

## The Solution

A **master template approach** -- one core data model and measure library (BI Controladoria), deployed as 9+ specialized variant dashboards. Each variant:

- Shares the same underlying data model and connection to UAU-based ERP (SQL Server)
- Inherits core measures (~140 shared DAX measures for bank balances, payment processes, closing status)
- Adds or filters for a specific financial domain (e.g., Juros e Multa focuses on penalty tracking, Suprimentos focuses on procurement)

This approach is complemented by:

- **BI Tesouraria (Treasury)**: A parallel treasury-focused model with 576 measures, emphasizing payment emission workflow and bank submission tracking
- **BI Fluxo Caixa (Cash Flow)**: A dedicated cash flow model with 186 measures for entry/exit/transfer reconciliation, quarterly comparisons, and working capital need analysis
- **BI Gestao de Tarefas**: A lightweight task management tracker

## Value Delivered

| Metric | Impact |
|--------|--------|
| **New SPE deployment** | Copy template + connect to new database = operational in hours, not weeks |
| **Consistent KPIs** | All SPEs report on the same definitions -- SaldoGerencial, Fechamento status, Juros e Multa, etc. |
| **Reduced maintenance** | Update the master model once, propagate to all variants |
| **Audit compliance** | Standardized closing status tracking (Fechamento Financeiro/Contabil) with SLA days |
| **Cash flow visibility** | Quarter-over-quarter variance using DATEADD(-4, QUARTER) for year-ago comparisons |

## Tech Stack

| Component | Technology |
|-----------|-----------|
| ERP | UAU-based ERP (SQL Server) |
| BI Platform | Power BI (Desktop + Service) |
| Reference Data | SharePoint |
| Measures | 1,357 DAX measures |
| ETL | Power Query M |

## The Key Innovation

The **template-to-variant pattern** is the central architectural decision. Rather than building monolithic dashboards or completely separate reports, this approach finds the middle ground: a shared core with domain-specific extensions. See [template-variants.md](template-variants.md) for the detailed breakdown of how this works in practice.
