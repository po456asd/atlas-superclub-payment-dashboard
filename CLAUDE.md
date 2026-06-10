# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-page construction payment dashboard for ATLAS Superclub Bangkok. Tracks contractor claims, payments, retention, and links scanned PDF/JPG evidence. Static site — no build step, no package manager.

## Run

```bash
python server.py          # serves repo on http://localhost:3000 with no-cache headers
```

Open `index.html` in browser directly also works (file:// will break PDF iframe links — use `server.py` for full functionality).

## Architecture

Single-page app: `index.html` (~1050 lines) + `data/contractors.json` + `data/payments.json`. No framework, no build, no tests.

Core data lives in JSON files under `data/`:

- **`data/contractors.json`**: contractor objects. Each has `contractor`, `ref`, `retention_total`, `retention_memo`, `retention_only` flag, and `contracts[]`. Each contract has financial columns `beforeVat`, `retention`, `vat`, `subTotal`, `whTax`, `grandTotal`, `paid`, plus `pdfUrl` pointing into `pdfs/` tree.
- **`data/payments.json`**: documented payment receipts keyed by `contractorRef`. This is THE source of truth for contractor totals — derived from `data/contractors.json` via `summarize()`.
- `contract.paid` per contract is a manual allocation record, NOT used for contractor totals.

Render loop calls `summarize(data, payments)` which returns `{ perContractor, grandTotals }`. Contractor-level totals always from `payments[]`. Overpaid badge when payments > claims.

Two top-level tabs: `Payment` and `Retention`. Retention tab uses `retention_total` / `retention_memo` from `contractors.json`; some entries are `retention_only:true` (no claim contracts).

PDF/evidence viewing: `showPDF(url,name)` opens modal with iframe; paths relative to repo root.

## Edits

When updating contractor amounts or adding claims:
1. Edit the `data` array entry — match the contractor by `ref`.
2. If a real payment came in, add/update entry in `payments` array (this drives the dashboard totals, not `contract.paid`).
3. CSV (`ATLAS_Superclub_Dashboard.csv`) and XLSX are exports, NOT inputs — regenerated via `exportExcel()` in browser. Don't hand-edit and expect the dashboard to update.
4. PDFs live under `pdfs/<N>.<CONTRACTOR>/`; retention memos under `pdfs/Retention/`. Reference relative path in `pdfUrl` / `retention_memo`.

Commit messages follow imperative style: see `git log` (e.g. "Update BKD Payment Claim No 09 retention amount").

## Gotchas

- `index.html` is too large for full Read tool dumps — use `Read` with `offset`/`limit`, or `Grep` to navigate.
- Excel export (`exportExcel`) and PDF export (`exportPDF`) use `summarize()` like the render loop — all three use the same data source.
- Negative `remaining` = overpaid contractor (e.g. MAXTECH, Big Ant). Code already styles these distinctly; don't "fix" the negative numbers.
