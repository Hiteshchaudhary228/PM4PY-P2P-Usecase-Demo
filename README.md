# Process Mining with pm4py: Procure-to-Pay (P2P) Analysis

An end-to-end process mining project using **Python and pm4py**. It starts from 16 related SAP-style tables, builds an event log from them, and produces process maps, variant and timing analysis, and a single-page HTML report.

> The dataset is **synthetic** (generated sample data), not real company data. The findings below describe this sample and illustrate what the tooling can do.

## Why this project

Commercial process mining platforms are powerful but often cost more than the savings they uncover for small and mid-sized companies. This project tests how much of that value an open-source stack can deliver:

- No licence fees
- Full control over the logic and the data
- Runs on a laptop, data stays local
- Easy to extend and automate

It is not a replacement for enterprise platforms (real-time monitoring, ready-made ERP connectors, dashboards at scale). It is a low-cost way to prove value first.

## Screenshots

| Frequency map | Performance map |
|---|---|
| ![Frequency DFG](images/original_frequency_dfg.png) | ![Performance DFG](images/original_performance_dfg.png) |

![Overview](images/1_overview.png)
![Variants](images/4_variants.png)

## Data model

Case = Purchase Order (`po_id`). About 600 POs, 3,948 events, 11 activities.

| Table | Contents |
|---|---|
| `LFA1_vendors`, `MARA_materials`, `T001W_plants`, `USERS` | Master data |
| `EBAN_purchase_requisitions` | Requisitions and approvals |
| `EKKO_po_header`, `EKPO_po_items` | Purchase orders |
| `MKPF_goods_receipt_header`, `MSEG_goods_receipt_items` | Goods receipts |
| `RBKP_invoice_header`, `RSEG_invoice_items` | Invoices and payment blocks |
| `PAYMENTS` | Payments |
| `CDHDR_change_header`, `CDPOS_change_items` | PO change log |
| `EVENT_LOG_derived` | Ready-made event log |

Tracing one PO: `EKKO` (po_id) -> `EKPO` (po_id, po_item) -> `MSEG` / `RSEG` (po_id, po_item) -> `RBKP` (invoice_id) -> `PAYMENTS` (invoice_id). All keys are listed in `RELATIONSHIPS.csv`.

## What the code does

1. Loads all CSV tables into a dictionary of DataFrames.
2. Builds the event log with **one function per activity** (`act_create_po`, `act_clear_invoice`, ...). Each returns the same standard columns, and an activity only creates a row if it actually happened.
3. Converts the log to pm4py format.
4. Discovers a frequency DFG and a performance DFG.
5. Analyses variants, case durations and resources.
6. Exports everything to a single-page HTML report (`build_report(event_log_df)`).

## Example findings (on the sample data)

- Only **28%** of 600 cases follow the standard path; there are **74** variants
- **6%** of POs skip the purchase requisition
- **81** invoices were blocked, 50 of them arriving before the goods receipt
- Invoice to payment is the longest step, about **35 days** on average
- Median case duration is **59 days**, maximum **114 days**

## Setup

1. Install Python 3.11+ and [Graphviz](https://graphviz.org/download/) (add it to PATH).
2. Create and activate a virtual environment:
   ```
   python -m venv venv
   venv\Scripts\activate
   ```
3. Install dependencies:
   ```
   pip install -r requirements.txt
   ```
4. Open the notebook, set the data folder path, and run the cells.

If Graphviz is not on PATH, set its location at the top of the report script (`DOT = r"C:\Program Files\Graphviz\bin\dot.exe"`) and in the Graphviz cell of the notebook.

## Ideas for further work

- Conformance checking against a designed P2P model
- Petri net and BPMN discovery (inductive and heuristics miners)
- Bottleneck analysis by vendor, plant or order value
- Resource and handover-of-work analysis
- Rework, late-payment and three-way-match violation detection
- Predictive monitoring (remaining time, outcome prediction)

## Tech stack

Python, pandas, pm4py, matplotlib, Graphviz


