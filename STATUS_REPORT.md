# STATUS REPORT — powerbi-financial-analytics

**Data:** 2026-04-18
**Executor:** Claude Code (instância do projeto powerbi-financial-analytics)
**Origem:** /home/vitormrodovalho/Documents/powerbi-financial-analytics
**Destino:** /home/vitormrodovalho/projects/powerbi-financial-analytics

## 1. Estado pré-migração
- Branch: main
- Commit HEAD: 3ad7ab5eed8e41884212c6fa8d2ff07df1bf9209 ("Initial commit")
- Dirty: 0 arquivos
- Unpushed: 0 commits (remote `origin/main` em sincronia com HEAD local)
- Tamanho: 6.0 MB

## 2. Git — ações executadas
- Commits criados: nenhum
- Push: não necessário — remote já em sincronia
- Branches empurradas: nenhuma

## 3. Dados externos (map-deps)
Referências encontradas a caminhos fora do projeto:

| Arquivo do projeto | Referência externa | Status |
|---|---|---|
| _(nenhuma)_ | _(n/a)_ | Nenhuma referência a `pbix-extraction/`, `Backup/pbix-*`, `Downloads/`, `Documents/Backup`, `OneDrive_2026*` ou paths absolutos em `/home/vitormrodovalho/` |

## 4. Mineração de dados (se aplicável)
- Status: N/A — o projeto não referencia diretamente pastas brutas
- Fontes consumidas: indeterminado isoladamente (ver seção 6)
- Pendências: nenhuma

## 5. Migração — integridade
- `cp -a` executado: sim
- `diff -qr` exit code: 0 (`/tmp/migration-diff-powerbi-financial-analytics.txt` 0 linhas)
- Verificação git pós-cópia: HEAD bate (`3ad7ab5…`)
- Origem deletada: sim

## 6. Sinalizações para consolidação (fase 3)

**Fora de escopo — decisão da conversa mestra**: as pastas abaixo existem mas não são referenciadas por este projeto:

- `~/Documents/pbix-extraction/` (1.2 GB) — provável fonte bruta comum aos 4 projetos `powerbi-*`
- `~/Documents/Backup/pbix-extracted/` (2.3 MB)
- `~/Documents/Backup/pbix-new/` (2.5 GB)

Recomendação: ver seção 6 dos reports de `powerbi-construction-analytics`, `powerbi-project-portfolio-management` e `powerbi-real-estate-bpo`. Decisão de delete só após consolidação cruzada dos 4 powerbi-*.

Arquivos externos que PODEM ser deletados com segurança (escopo deste projeto): nenhum.

Arquivos externos que NÃO PODEM ser deletados ainda: nenhum decidido aqui.

## 7. Problemas encontrados
Nenhum.

## 8. Confirmação final
- [x] Projeto funcional em `~/projects/powerbi-financial-analytics/`
- [x] Git remoto em dia
- [x] Origem removida
- [x] Report pronto para envio à conversa mestra
