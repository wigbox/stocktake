# Wig Stocktake

A mobile web app for counting wig stock in the store. Staff scan each wig's barcode (or type its 5-digit number) on their phone; the app checks the count against a copy of the stock list frozen at the start of the count, and reports what is missing, what was found but is recorded as not in stock, and what is unknown.

**App link:** https://wigbox.github.io/stocktake/
Interface in Hebrew. Works on iPhone and Android (Safari / Chrome — not inside in-app browsers such as Cliq).

## How it fits together

```
  Master stock sheet (Google Sheets)
        │  IMPORTRANGE — column A (wig number) and column N (status) only
        ▼
  "Wig Stocktake" Google Sheet  ◄──►  Apps Script (attached to that sheet)
                                            ▲
                                            │  saves scans, checks PINs, builds reports
                                            │
                          index.html in this repository (GitHub Pages)
                                  = the screens staff open on their phones
```

| Part | Where it lives | What it does |
|---|---|---|
| **index.html** | This repository (GitHub Pages) | The app screens: sign-in, camera scanning, typing, manager screen, reports, Excel download |
| **Apps Script** (`Code.gs`) | Attached to the *Wig Stocktake* sheet (Extensions → Apps Script). Not stored here. | Checks PINs, saves every scan, prevents double counting, freezes the stock list, builds reports, writes the Summary tab |
| **Wig Stocktake sheet** | Google Drive | All stocktake data: settings, frozen list, scans, results, history |
| **Master stock sheet** | Google Drive | The live stock list. The app never opens it directly — only columns A and N are copied in via IMPORTRANGE |

The camera can't be used inside an Apps Script page on iPhone, which is why the screens are hosted here and the script is used only as a back end.

## The Wig Stocktake sheet — tabs

| Tab | Contents | Edited by |
|---|---|---|
| **Summary** | Current/last count: totals, scans per person, list of wigs to check | Script (auto) |
| **Settings** | Staff PIN, Manager PIN, each status's group and Hebrew name | Manager |
| **Import** | IMPORTRANGE of master columns A (wig number) and N (status) | Formulas only |
| **Stocktakes** | One row per count: type, opened/closed, by whom, totals | Script |
| **Snapshot** | Frozen copy of Import for the open count | Script |
| **Scans** | Each wig counted in the current count (time, who, scanned/typed, device) | Script |
| **Duplicates** | Repeat scans of a wig already counted | Script |
| **Results** | Per-wig result of every closed count (history) | Script |
| **Scan log** | Every scan, duplicate and removed scan of every closed count | Script |

Don't edit Snapshot, Scans or Duplicates by hand.

## Status groups (set in the Settings tab)

| Group | Statuses (default) | Meaning in the count |
|---|---|---|
| In store | In Stock, Inventory Reserved | Expected on the shelf — not scanned = **missing** |
| Consignment | Consignment | Counted in an *inventory* count, excluded from an *accounting* count |
| Fix-Return | Fix-Return | Owned but away for repair — listed separately |
| Not in store | Sold, Returned, Personal Use, damaged, Ordered, Order Reserved, Wish List, Barter, Missing, Cancelled, Other | Scanned = **problem** (in the store but recorded as not in stock) |

## How a count works

1. **Manager opens a count** — chooses Inventory or Accounting, checks the import, then freezes it. The Import tab is copied to Snapshot; this frozen list is what the count is compared against, even if the master sheet changes during the count.
2. **Staff scan** — each wig is saved once with date, time, who, scanned or typed, and device. Staff see only "נקלט ✓" (captured) or "כבר נספרה" (already counted). Scans queue on the phone if the connection drops and save automatically when it returns.
3. **Manager follows progress** — live totals and reports in the app (manager PIN), or the Summary tab in the sheet.
4. **Manager closes the count** — results are saved to Results and Scan log, and the Summary tab is updated.

## Reports

Scanned but not in stock · Missing (with a flag if the status changed during the count) · Unknown numbers · Fix-Return · Consignment · Typed by hand (possible barcode problems) · Duplicate scans. All downloadable to Excel; wig lookup shows a wig's current status and its history across counts.

## Access and privacy

- Two PINs: **staff** (scanning only) and **manager** (opening/closing counts, reports, lookup, undo). PINs are checked by the Apps Script, never in this page's code.
- The script is restricted to its own spreadsheet (`@OnlyCurrentDoc`) and cannot open the master sheet.
- Only wig numbers and statuses reach the app. Customer details never leave the master sheet.
- Never put PINs, passwords or customer data in this repository — it is public.

## Updating

- **Screens:** upload a new `index.html` here (Add file → Upload files → Commit). Live within about a minute.
- **Script:** paste the new code into Apps Script → Save → Deploy → Manage deployments → ✏️ → New version → Deploy. The link stays the same.
- PINs and status groups: change them in the Settings tab — no update needed.
