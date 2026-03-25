# Template-Variant Pattern

## What Is the Template-Variant Pattern?

The template-variant pattern is an architectural approach to Power BI deployment where a single **master** report (the template) serves as the source for multiple **variant** dashboards, each specialized for a specific financial domain. Rather than building separate reports from scratch, each variant is derived from the master model and inherits its core measures, data connections, and data model.

This is the key innovation of this financial analytics suite.

## How Does One Power BI Model Become 9 Dashboards?

The BI Controladoria master contains **594 DAX measures** organized into **10 display folders** (functional domains):

| Display Folder | Measure Count | Domain |
|---------------|---------------|--------|
| Banco Gerencial | ~5 | Bank balance monitoring (LASTNONBLANK semi-additive) |
| Processos de Pagamento | ~12 | Payment process type tracking (Cotacao, Compra Rapida, etc.) |
| Variacoes | ~8 | Month-over-month and percentage variances |
| Fechamento Financeiro | ~15 | Financial closing status, SLA days, open/closed counts |
| Fechamento Contabil | ~13 | Accounting closing status, parallel to financial |
| Pagamentos Fora do Prazo | ~4 | Late payment averages (emission-to-payment, emission-to-due-date) |
| Juros e Multas | ~3 | Penalty and late fee totals and counts |
| Extrato Auditoria | ~7 | Bank reconciliation (UAU balance vs. bank statement) |
| Emissao Pagamentos | ~10 | Payment emission workflow (overdue, upcoming, sent-to-bank) |
| Mapa | ~7 | Cash position map with closing dates |

Each variant dashboard is created by:

1. **Copying** the master `.pbix` file
2. **Removing pages** not relevant to the variant's domain
3. **Keeping the full data model** and all measures (Power BI does not penalize unused measures)
4. **Adjusting the landing page** and navigation to focus on the variant's specialty
5. **Optionally adding** variant-specific visuals or page-level filters

Because all measures remain in the file, any variant can still reference any core measure. This means a "Juros e Multa" variant can still show a bank balance card if needed.

## The 9 Known Variants

| # | Variant Name | Specialization | Key Measures Used |
|---|-------------|----------------|-------------------|
| 1 | **Bancos Gerenciais** | Bank balance overview across all SPE accounts | SaldoGerencial (LASTNONBLANK), SaldoGerencialHistorico, SaldoGerencialAtual |
| 2 | **Conciliacao** | Bank reconciliation -- comparing UAU ERP balances against actual bank statements | StatusConciliacao, SaldoFinalUAU, ValorFinalExtrato, DiferencaSaldoExtrato, QtdConciliado, QtdNaoConciliado |
| 3 | **Contas Pagas** | Paid invoices tracking -- what was paid, when, and by which process type | Contas a Pagar, Contas Recebidas, SomaValorParc, payment process type breakdowns |
| 4 | **Fechamentos** | Financial and accounting closing status dashboard -- which SPEs are closed, open, or late | AbertoFechFin, FechadoFechFin, DiasFecharento, UltimoStatus, QtdFechamentos, MediaFechamentos |
| 5 | **Juros e Multa** | Penalty and late fee analysis -- how much was paid in penalties, which suppliers cause the most | PagamJurosMultas, QtdJurosMultas, PagamAcumulado |
| 6 | **Juros e Multa 1** | Alternative view of the same penalty data, possibly with different time granularity or filtering | Same Juros e Multas measures with different page-level filters |
| 7 | **Suprimentos** | Procurement and supply chain -- payment processes filtered to procurement-related types | TotalPagamentos, Cotacao, ProcessoPagamento, CompraRapida, ProcessoVinculado, SomaValorParc |
| 8 | **Fechamento Gerencial** | Management-level closing summary -- aggregated view across all SPEs with KPI indicators | MediaFechamentos, MediaAberturas, StatusFechamento, MediaDiasFechamento |
| 9 | *(BPO variants)* | BPO-specific versions exist (BPO - BI Controladoria) that add BPO-provider-level aggregation across multiple client SPEs | Same core measures, additional BPO-level filtering |

## Shared Core vs. Variant-Specific Measures

### Shared Core (~140 measures)

These measures exist in every variant because they are part of the master model:

- **Banco Gerencial**: SaldoGerencial, SaldoGerencialHistorico, SaldoGerencialAtual
- **Processos de Pagamento**: TotalPagamentos, Cotacao, ProcessoPagamento, CompraRapida, ProcessoVinculado, ProcessoTransporte, Medicao, AcompEntradaSemEstoque, CompPatrimonio, AdiamCaixaObra, Outros, SomaValorParc
- **Variacoes**: YTD Mes, YTD Mes Anterior, VariacaoMesAnterior, VariacaoPercMesAnterior, PercForaPrazo, Soma Total, DiasFechamento, teste
- **Fechamento Financeiro**: AbertoFechFin, FechadoFechFin, Ultimo Status, Ultima Data Alteracao, Dias Fechamento, QtdFechamentos, ContUltStatusFechado, ContUltStatusAberto, SemStatus, QtdAberturas, Ultima Data Fechamento, StatusFechamento, Media de Dias Fechamento, MediaFechamentos, MediaAberturas
- **Fechamento Contabil**: Ultima Data Alteracao, Ultimo Status, Dias Fechamento, AbertoFechCon, QtdFechamentos, FechadoFechCon, QtdAberturas, SemStatus, Ultima Data Fechamento, StatusFechamento, MediaFechamentos, MediaAberturas, Media de Dias Fechamento
- **Pagamentos Fora do Prazo**: Media Emissao-Pagamento, Media Emissao-Vencimento, Media Sistema-Vencimento, Quantidade
- **Juros e Multas**: PagamJurosMultas, QtdJurosMultas, PagamAcumulado
- **Extrato Auditoria**: StatusConciliacao, SaldoFinalUAU, ValorFinalExtrato, QtdConciliado, QtdNaoConciliado, DiferencaSaldoExtrato
- **Emissao Pagamentos**: ProcessosEmAtraso, ProcessosAVencer, ContagemClusterProcesso, SomaValorPagamento, QtdFornecedor, SomaProcessosAtraso, SomaProcessosAVencer, ProcessosEmEmissao, EnviadosABanco, NaoEnviadosABanco
- **Mapa**: FormatacaoDataSaldo, ContDataSaldo, Ultimo Mapa, Data Ultimo Mapa, Dias Para Fechamento Mapa, Qtd Mapas Processados, Media Dias Fechamento
- **Contas a Pagar / Recebidas**: Contas a Pagar, Contas Recebidas, Qtd Contas a Pagar, Qtd Contas Recebidas

### Variant-Specific

Variants do not typically add new DAX measures. Instead, they specialize through:

- **Page visibility**: Only pages relevant to the variant's domain are shown
- **Page-level filters**: Pre-applied filters narrow the data to the relevant domain
- **Visual layout**: Charts and tables are arranged to highlight variant-specific KPIs
- **Navigation**: Bookmarks and buttons guide users to the variant's focus area

## How to Deploy a New Variant

1. **Copy the master file**: Duplicate `BI_Controladoria.pbix` and rename (e.g., `BI_Controladoria_-_NovaVariante.pbix`)
2. **Open in Power BI Desktop**: All data sources, measures, and pages load as-is
3. **Delete irrelevant pages**: Remove report pages that are not needed for this variant's focus area
4. **Set the landing page**: Configure the first page to match the variant's primary use case
5. **Apply page-level filters** (optional): Add default filters to narrow the displayed data
6. **Customize visuals** (optional): Rearrange or add visuals specific to this variant
7. **Change the data source** (for a new SPE): Update the SQL Server connection parameters to point to the new SPE's database
8. **Publish**: Deploy to Power BI Service in the appropriate workspace

## Maintenance Strategy

The maintenance workflow follows a **master-first** approach:

1. **All measure changes happen in the master** (`BI_Controladoria.pbix`)
2. **Test in the master**: Validate the formula changes with test data
3. **Propagate to variants**: For each variant, either:
   - Re-derive from the updated master (preferred for major changes)
   - Manually copy the changed measure (for minor fixes)
4. **Version control**: The DAX measures in this repository serve as the single source of truth for measure definitions, independent of any specific `.pbix` file

This approach ensures consistency: every variant uses the same formula for `SaldoGerencial`, the same closing status logic, and the same penalty calculations. When a formula needs updating, it is changed once and propagated, rather than being fixed independently in 9+ files.
