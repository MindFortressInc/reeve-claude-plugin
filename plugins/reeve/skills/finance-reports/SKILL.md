---
name: finance-reports
description: "Pulls QuickBooks reports from Reeve: profit and loss for a period, accounts receivable aging (who owes the business money, by days overdue) and accounts payable aging (what the business owes its vendors). Covers one company or every connected company with a consolidated total. Use when someone asks for a P&L, income statement, net income, margins for a period, an AR or AP aging, who owes us money, overdue invoices, or what we owe vendors."
---

# QuickBooks reports through Reeve

Reeve reads the QuickBooks companies the user has connected in Reeve. It
never writes to QuickBooks.

## Tools

| Tool | What it returns |
| :- | :- |
| `finance_list_entities` | The connected QuickBooks companies, with currency, accounting basis, fiscal-year start and consolidation groups |
| `finance_get_pnl` | Profit and loss for a period: revenue, COGS, gross profit, expenses and net income |
| `finance_get_ar_aging` | Accounts receivable aging as of a date, by customer and days overdue |
| `finance_get_ap_aging` | Accounts payable aging as of a date, by vendor and days overdue |

## Workflow

1. **Pick the company.** If the user names a company, a group or a country
   ("the UK business"), pass it as `entity`. If they don't and more than one
   company could be meant, call `finance_list_entities` and ask which one,
   or leave `entity` out to get every company plus a consolidated total.
2. **Pick the period or date.**
   - P&L: pass `period` in words ("last month", "Q3", "YTD", "TTM",
     "FY2026", "2026-08"), or `start` and `end` dates. Use
     `granularity` ("month", "quarter", "year") for a column per period, and
     `basis` ("accrual" or "cash") only when the user asks for one.
   - Aging: pass `as_of` ("end of last month" or a date). It defaults to today.
3. **Call the report tool** (`finance_get_pnl`, `finance_get_ar_aging` or
   `finance_get_ap_aging`).
4. **Answer from the result.** State the period, basis and company from the
   result's `provenance`. For an aging report, lead with the total, then the
   largest and most overdue balances. For a P&L, lead with revenue and net
   income, then the lines that changed the most.

If the result lists products instead of a report, the user has more than one
Reeve product with books: ask which one and pass it as `product`. If no
company is connected, tell the user to connect QuickBooks in the Reeve app.
