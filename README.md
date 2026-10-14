# Project 1: Executive Revenue & Profitability

**Why is profit not keeping pace with growth?** A year-over-year performance review for PayBridge NG, a fictional Nigerian payments company.

*Project 1 of 9 built on the PayBridge dataset. Tools: SQL Server, Python, Power BI.*

<img width="1600" height="820" alt="Executive Revenue   Profitability" src="https://github.com/user-attachments/assets/74d4a601-8f03-49a7-8835-162143d383bb" />


<img width="908" height="521" alt="Screenshot 2026-10-13 140052" src="https://github.com/user-attachments/assets/470a89c8-1362-4a43-ab14-a08cb2d01fa3" />
<!-- Add your finished dashboard screenshot to the images folder as dashboard.png -->


---------------------------------
> **About this project.** PayBridge NG is a fictional company and all data is synthetic. Some patterns were deliberately built into the simulation so the analysis has something to find. The purpose is to demonstrate the method: framing a business problem, building a trustworthy dataset, analysing it, and recommending action.

---

## Headline finding

Fee revenue grew **22.1% to ₦144.0M**, but net revenue grew only **5.2% to ₦51.1M**. Net margin fell from **41.2% to 35.5%**.

Had the 2024 margin held, 2025 net revenue would have been about **₦59.3M**, which is **₦8.2M** more than the business actually kept.

---

## 1. Problem and context

PayBridge NG processes merchant payments through POS, web checkout, mobile app, USSD and QR channels. Management sees strong top-line growth, but profit is not following, and reporting is built around revenue totals that hide where the margin goes.

**Primary question:** where does revenue growth stop converting into profit, and what should management change first?

**Stakeholders:** CEO and CFO (growth quality), Head of Product (channel and product margin), Head of Sales (pricing and merchants), Head of Operations (reliability and cost).

**Decisions the dashboard supports:**
1. Which channels, products and merchants to grow, reprice or deprioritise
2. Which cost lines to attack first
3. Which markets deserve further investment

The full problem statement, with analytical questions, KPIs and six hypotheses, is in `docs/PayBridge_NG_Problem_Statement_Brief.pdf`.

## 2. Data

| Item | Detail |
|---|---|
| Source | Synthetic, generated with a configurable Python simulation (fixed seed, reproducible) |
| Period | January 2024 to December 2025 |
| Scale | About 1 million transaction attempts, 400 merchants, 14 cities |
| Model | Star schema in SQL Server: one transaction-level fact table plus dimensions (merchant, product, channel, location, bank, response code, pricing plan, terminal, date) and a pricing-history bridge |
| Documentation | Data dictionary in `docs/` |

## 3. Exploration and cleaning

The data is generated, so there was no raw cleaning. The work was making sure the build could be trusted:

- **Constraints in the database.** Primary keys, foreign keys and check constraints reject bad rows (for example, a discount larger than the fee it applies to).
- **A constraint caught a flaw in my own generator.** The first load failed because some merchants had a pricing period that ended before it started. I fixed the logic, regenerated the data and reloaded.
- **Load checks.** Loaded with constraint checking on, then compared row counts across all 13 tables.
- **Reconciliation.** Yearly revenue and cost totals in SQL Server were compared with the generator's own totals. <!-- keep this line only if you ran validation block 3 -->
- **Decision:** the optional database indexes were skipped after the build stalled for lack of memory. Power BI imports whole tables, so they were not needed.
- **Definition to note:** transaction counts include retries, since each attempt is a row in the data.

## 4. Analysis

Each hypothesis from the problem brief was tested against the data (2025 compared with 2024).

| # | Hypothesis | Evidence | Verdict |
|---|---|---|---|
| H1 | Growth is shifting to lower-margin channels | USSD rose from 12% to 18% of attempts and earns the lowest margin (15%). POS fell from 43% to 36%. | Supported, but not the whole story |
| H2 | Costs rose faster than fees | Cost per successful transaction rose 9% (₦157.2 to ₦171.9) while fee per transaction stayed flat (₦280.2 to ₦281.2) | **Supported** |
| H3 | Discounts are eroding take rate | Discounts given rose from ₦10.7M to ₦18.0M (+68%); take rate fell from 0.98% to 0.94% | Supported |
| H4 | Failed transactions drag profit | Failure rate rose from 6.1% to 7.2%; fees forgone rose from ₦6.9M to ₦9.9M; cost on failed attempts rose from ₦2.8M to ₦4.3M | Supported |
| H5 | New cities are below break-even | 2025 net margin was 43.0% in new cities against 35.5% elsewhere | **Not supported** |
| H6 | Festive season squeezes margin | Nov to Dec margin was 33.1% against 36.1% for Jan to Oct | Weakly supported |

## 5. The dashboard

A one-page Power BI report answers the questions in the order a CFO would ask them: how big is the change, when did it happen, where is the profit going, and who are we relying on.

**How to read it:**
- **KPI cards** show the year's value and change against the prior year
- **Monthly trends** compare fee revenue and net margin with the prior year
- **Channel margin** shows where profit is thinnest
- **Net revenue waterfall** shows which channels added or removed profit
- **Top 10 merchants** shows concentration

DAX measures and the colour theme are in the `powerbi/` folder.

## 6. Key insights

1. **Costs grew faster than revenue.** Total transaction cost rose 34.1% (₦69.3M to ₦92.9M) against revenue growth of 22.1%. Active merchants rose 19.1% to 392 while transactions rose 23.0%, so most of the volume growth came from signing more merchants.
2. **Every channel lost margin, so mix is only part of the cause.** POS fell from 44% to 38%, Web from 31% to 24%, App from 62% to 59%, USSD from 19% to 15%, QR from 51% to 47%. The across-the-board decline points to cost and pricing, with the shift toward USSD making it worse.
3. **Costs rose but prices did not.** Per-transaction fees were flat while per-transaction costs rose 9%.
4. **Discounts deepened.** The 68% rise in discounts outpaced growth, and take rate fell about 4%. Falling concentration (top 10 merchants went from 46% to 42% of net revenue) is not all good news: it is consistent with the largest merchants receiving deeper discounts.
5. **Failures cost money twice.** Each failed attempt forgoes a fee and still incurs cost.
6. **Expansion is not the problem.** New cities show healthier margins than established ones, although they are small and young.
7. **Enterprise merchants earn half the profit at the lowest margin.** Enterprise provides ₦25.7M (50%) of net revenue at a 29% margin, against 45% for Mid-market and 52% for SME. Its net revenue fell 7% while those two segments grew 20% to 22%.
8. **Growth came from the mobile app, while POS and Web lost ground.** Net revenue rose ₦2.5M overall: the app added ₦4.1M, with smaller gains from USSD (₦0.5M) and QR (₦0.3M), but POS lost ₦1.6M and Web ₦0.6M.
9. **Two merchants carry a large share of the profit.** The largest two earn ₦11.9M, 23% of net revenue, and the top 10 earn 42%.
10. **Micro merchants lose money on every transaction.** Their net margin is -15%. The amount is small (about ₦0.1M), but it shows the flat fee does not cover the cost of a transaction.

## 7. Recommendations

| Priority | Action | Estimated value | Basis |
|---|---|---|---|
| 1 | **Renegotiate processing and switching costs** | About ₦2.8M a year | Processing is ₦56M, about 60% of all cost, so a 5% cut in processing alone is worth about ₦2.8M. Recovering a third of the 9% per-transaction cost increase (about 3% of total cost) gives the same figure |
| 2 | **Tighten discount governance**: approval thresholds for deep discounts, starting with a review of Enterprise terms (50% of net revenue at a 29% margin) | About ₦3.6M a year | Recovering 20% of ₦18.0M in discounts. Holding take rate at the 2024 level would be worth roughly ₦5M to ₦8M |
| 3 | **Reprice USSD and steer users to the app**: add a minimum fee or pass through session costs | About ₦0.9M a year | USSD fee revenue was ₦9.3M in 2025 at a 15% margin. Each 10 points of margin gained is worth about ₦0.9M, and reaching a 35% margin would be worth about ₦1.9M |
| 4 | **Cut the failure rate back to the 2024 level** | About ₦2M a year | About 15% fewer failures, applied to ₦9.9M of fees forgone and ₦4.3M of failure cost |
| 5 | **Plan for the festive season**: seasonal pricing and cost controls | Not sized | Margin runs about 3 points lower in Nov to Dec |
| 6 | **Keep investing in new cities**, and monitor margin as they scale | n/a | Margins are currently healthy |
| 7 | **Fix Micro merchant pricing**: raise the flat fee or add a minimum | About ₦0.1M a year | Micro merchants run at a -15% margin, so break-even recovers the current loss. Small in naira, but it is a pricing flaw |

**Reading the numbers:** these are sizing estimates, not forecasts. The effects overlap, so they should not be simply added together. Even so, the first four actions together are of the same order as the ₦8.2M margin gap.

## 8. Limitations

- Synthetic and simplified: no customer table, refunds, chargebacks, operating expenses or taxes, and one currency
- Costs are recorded per transaction, whereas real costs often arrive as invoices and are allocated later
- Two years of data, so seasonality and trend are hard to separate
- The analysis shows what changed and where, not proof of cause. Estimates rest on the stated assumptions
- New-city results rest on small volumes and short history


**Author:** Sanya Ajibola Ebenezer | [LinkedIn](https://www.linkedin.com/in/ajibola-sanya-119a91384/) <!-- add your link -->
