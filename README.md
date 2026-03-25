# Power BI Financial Analytics Suite

**4 dashboards for financial controlling, treasury, and cash flow management**

Author: **Vitor Rodovalho**

---

## Overview

This repository contains the extracted knowledge artifacts (DAX measures, Power Query M code, data models, and relationships) from a production Power BI financial analytics suite built for a Brazilian real estate development and BPO financial operations environment.

### Key Highlights

- **1-master to 9-variants pattern**: One core Controladoria (Financial Controlling) model is deployed as 9+ specialized dashboards -- Bancos Gerenciais, Conciliacao, Contas Pagas, Fechamentos, Juros e Multa (x2), Suprimentos, Fechamento Gerencial, and more. See [docs/template-variants.md](docs/template-variants.md) for the full breakdown.
- **1,357 total DAX measures** across 4 dashboards
- **Semi-additive bank balance measures** using `LASTNONBLANK` for point-in-time financial snapshots
- **Complete cash flow model** with entry/exit/transfer reconciliation and quarterly comparisons via `DATEADD`
- **Connected to UAU-based ERP** (SQL Server), with SharePoint for reference data

---

## Dashboard Inventory

| Dashboard | Measures | Purpose | Complexity |
|-----------|----------|---------|------------|
| BI Controladoria (Financial Controlling, Master) | 594 | Financial controlling -- bank balances, payment processes, closing status, reconciliation, penalties | High |
| BI Tesouraria (Treasury) | 576 | Treasury management -- mirrors Controladoria (Financial Controlling) with treasury-specific payment emission and supplier tracking | High |
| BI Fluxo Caixa (Cash Flow) | 186 | Cash flow -- bank balances, entries, exits, transfers, quarterly variance, working capital need | Medium |
| BI Gestao de Tarefas | 1 | Task and project management -- simple row count for task tracking | Low |

---

## Repository Structure

```
powerbi-financial-analytics/
├── README.md
├── LICENSE
├── ANONYMIZATION_RULES.md
├── .gitignore
├── docs/
│   ├── architecture.md           # System architecture and DAX patterns
│   ├── business-case.md          # Industry context and value proposition
│   ├── dashboard-catalog.md      # Detailed catalog of all 4 dashboards
│   └── template-variants.md      # The 1-master to 9-variants pattern
├── dax/
│   ├── controladoria-master/     # BI Controladoria (594 DAX measures)
│   │   ├── measures.md
│   │   └── columns.csv
│   ├── tesouraria/               # BI Tesouraria (576 DAX measures)
│   │   ├── measures.md
│   │   └── columns.csv
│   ├── fluxo-caixa/              # BI Fluxo Caixa (186 DAX measures)
│   │   ├── measures.md
│   │   └── columns.csv
│   └── gestao-tarefas/           # BI Gestao de Tarefas (1 DAX measure)
│       ├── measures.md
│       └── columns.csv
├── power-query/
│   ├── controladoria.md
│   ├── tesouraria.md
│   ├── fluxo-caixa.md
│   ├── gestao-tarefas.md
│   ├── etl-patterns/
│   └── connections/
├── data-model/
│   ├── schema/
│   │   ├── controladoria.csv
│   │   ├── tesouraria.csv
│   │   ├── fluxo-caixa.csv
│   │   └── gestao-tarefas.csv
│   └── relationships/
│       ├── controladoria.csv
│       ├── tesouraria.csv
│       ├── fluxo-caixa.csv
│       └── gestao-tarefas.csv
└── assets/
    └── screenshots/
```

---

## How to Use

1. **Browse DAX measures**: Open `dax/{dashboard}/measures.md` to see all DAX measures organized by display folder (functional domain).
2. **Understand the data model**: Check `data-model/schema/` for table/column definitions and `data-model/relationships/` for the relationship graph.
3. **Review Power Query ETL**: The `power-query/` folder contains the M code for all data sources and transformations.
4. **Learn the architecture**: Start with [docs/architecture.md](docs/architecture.md) for the system design, then [docs/template-variants.md](docs/template-variants.md) for the unique template-to-variant deployment pattern.

---

## Tech Stack

- **Power BI** (Desktop and Service)
- **SQL Server** -- UAU-based ERP (primary data source)
- **SharePoint** -- reference data and document storage
- **DAX** -- 1,357 measures for financial calculations
- **Power Query M** -- ETL transformations and data connections

---

## Anonymization

All sensitive information (IP addresses, company names, personal identifiers, tenant URLs) has been replaced with generic placeholders using an automated sanitization script. See [ANONYMIZATION_RULES.md](ANONYMIZATION_RULES.md) for the full replacement table. The original `.pbix` files are excluded from this repository.

---

> **Note:** Original DAX and Power Query (M) code preserves Portuguese identifiers as-is from the source Power BI models.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
