<h1 align="center">Lotus Group Retail Analytics: Business Insights</h1>

# Purpose

This document summarises what the analysis found, why it matters, and what Lotus Group management should do about it. It sits on top of the Power BI report in [`05_power_bi/`](../05_power_bi/) and uses the measures defined in [`DAX_measures_catalog.md`](../04_DAX/DAX_measures_catalog.md).

All figures cover **1 January 2022 – 31 December 2024** and are in **Egyptian pounds (EGP)** unless stated otherwise.

---

# Executive summary

| KPI | Value |
|---|---|
| Total Sales | EGP 45.35M |
| Gross Profit | EGP 9.70M |
| Gross Margin % | 21.4% |
| Total Orders | 12,000 |
| Units Sold | 43,313 |
| Average Order Value | EGP 3,779 |
| Customers who purchased | 2,945 |
| Returned Amount | EGP 4.01M (8.8% of sales) |
| Return Rate % (orders) | 8.8% |
| Employee Attribution Rate % | 94.7% |

**Five things management should know:**

1. **Clothing earns more profit than electronics, even though electronics sells three times as much.**
2. **Three Greater Cairo flagship stores produce 37% of all sales.**
3. **The top 20% of customers drive 59% of revenue, but the loyalty tiers don't identify them.**
4. **Ramadan lowers electronics sales by 19% a day, while clothing sales hold steady.**
5. **37% of returned value comes from preventable process errors.**

---

# 1. Sales performance over time

| Year | Total Sales | YoY Growth | Gross Margin |
|---|---|---|---|
| 2022 | EGP 15.25M | – | 20.9% |
| 2023 | EGP 14.52M | −4.8% | 22.0% |
| 2024 | EGP 15.58M | +7.3% | 21.3% |

**What we see**
- Sales fell in 2023 and recovered in 2024, ending slightly above 2022.
- Gross margin stayed within a narrow band (20.9–22.0%), so the 2023 dip came from lower volume, not from heavier discounting or rising costs.

**What it means**
- The business is broadly stable. The 2024 recovery is real but modest. Over the full three years, sales grew only about 2% in total.

---

# 2. Product and category performance

| Category | Sales | Share of Sales | Units | Share of Units | Gross Margin | Gross Profit | Share of Profit |
|---|---|---|---|---|---|---|---|
| Electronics | EGP 34.39M | 75.8% | 4,686 | 10.8% | 13.1% | EGP 4.50M | 46.4% |
| Clothing | EGP 10.96M | 24.2% | 38,627 | 89.2% | 47.4% | EGP 5.20M | 53.6% |

**What we see**
- Electronics dominates revenue but earns a thin margin. Clothing dominates unit volume and earns a margin more than three times higher.
- **Clothing generates 53.6% of gross profit from 24.2% of sales.**
- The top subcategories by sales are all electronics: Mobile Phones (16.1%), Laptops (15.8%), TVs & Screens (13.3%) and Gaming (10.6%).
- The top 10 products (out of 345) produce 40% of sales. All ten are electronics, led by the iPhone 14 128GB, Dell Inspiron 15 3000 and HP Laptop 15.
- Average discount is about 7.9% in both categories, so the margin gap comes from pricing and cost, not from discounting.

**What it means**
- Looking at revenue alone makes electronics seem like the core of the business. Looking at profit, clothing is just as important.
- Electronics revenue is concentrated in a small number of high-ticket products, which makes it exposed to supplier pricing and stock-outs on those lines.

---

# 3. Store and geographic performance

| Region | Share of Sales |
|---|---|
| Greater Cairo | 36.8% |
| Alexandria | 18.4% |
| Nile Delta | 16.3% |
| Canal Zone | 14.3% |
| Upper Egypt | 6.2% |
| Red Sea | 5.0% |
| South Sinai | 2.9% |

| Store Type | Stores | Total Sales | Average Sales per Store | Share of Sales |
|---|---|---|---|---|
| Flagship | 3 | EGP 16.71M | EGP 5.57M | 36.8% |
| Standard | 5 | EGP 15.40M | EGP 3.08M | 34.0% |
| Small | 7 | EGP 13.24M | EGP 1.89M | 29.2% |

**What we see**
- The three flagships (Cairo Festival City, City Stars and Mall of Egypt) are the top three stores and together produce 36.8% of sales.
- Each Small store averages about one third of a flagship's revenue.
- The weakest stores are Luxor Mall (2.3% of sales), Sharm Plaza (2.9%) and Ismailia Festival (3.5%).
- On the Executive Overview, *average sales per transaction line* is highest in South Sinai and lowest in Upper Egypt. Average basket value varies far less between stores than total sales does, so the gap between stores comes mainly from the number of orders, not from basket size.

**What it means**
- Performance depends heavily on Greater Cairo. Any disruption at the flagships would have an outsized effect on group results.
- Smaller stores aren't failing on basket value. They're handling fewer orders. That points to footfall and traffic (location, marketing) rather than in-store selling.

---

# 4. Customer performance

**What we see**
- 2,945 of the 3,000 registered customers made at least one purchase.
- **The top 20% of customers generate 59.3% of revenue**, and the top 10% generate 38.6%.
- Average spend per customer is almost the same in every loyalty tier:

| Loyalty Tier | Customers | Sales | Share of Sales | Sales per Customer |
|---|---|---|---|---|
| Bronze | 1,517 | EGP 22.57M | 49.8% | EGP 14,876 |
| Silver | 866 | EGP 13.90M | 30.7% | EGP 16,054 |
| Gold | 432 | EGP 6.91M | 15.2% | EGP 15,997 |
| Platinum | 130 | EGP 1.97M | 4.3% | EGP 15,166 |

**What it means**
- Revenue depends heavily on a relatively small group of customers.
- The loyalty tiers aren't separating high-value customers from the rest. A Platinum customer spends roughly the same as a Bronze one, and half of all revenue comes from Bronze members. The tiers are probably based on something other than spend (for example, tenure or registration date).

---

# 5. Employee attribution

**What we see**
- 632 of 12,000 orders (5.3%) have no employee recorded. In 2024 this was 205 orders, giving a 94.95% attribution rate.
- Every recorded employee ID matches a valid employee in `dim_employees`, so the problem is that IDs weren't captured, not that wrong IDs were entered.
- Cashiers are the largest staff group (80 of 216 employees) and account for the largest share of attributed sales.

**What it means**
- At 95% attribution, employee-level reporting is usable but not complete. It isn't yet reliable enough to base commissions or individual performance reviews on.
- The employee field records *who processed the order*. It shouldn't be read as *who made the sale*, which is why cashiers rank highest.

---

# 6. Seasonality

| Period | Days | Average Daily Sales | Difference |
|---|---|---|---|
| Non-Ramadan | 1,006 | EGP 41,856 | – |
| Ramadan | 90 | EGP 36,047 | **−13.9%** |
| Weekday (Sun–Thu) | 783 | EGP 42,688 | – |
| Weekend (Fri–Sat) | 313 | EGP 38,104 | **−10.7%** |

**Ramadan by category (average daily sales)**

| Category | Non-Ramadan | Ramadan | Difference |
|---|---|---|---|
| Electronics | EGP 31,879 | EGP 25,800 | −19.1% |
| Clothing | EGP 9,977 | EGP 10,247 | +2.7% |

**What we see**
- Daily sales are lower during Ramadan, and **all of that drop comes from electronics**. Clothing sales rise slightly.
- Weekends are quieter than weekdays, which is unusual for retail.

**What it means**
- Ramadan is a period to promote clothing (including Eid purchases), not electronics. Electronics campaigns are likely to do better just before or after Ramadan.
- Weaker weekend sales point to an opportunity to drive weekend footfall, or to a store-hours and staffing pattern that's worth reviewing.

> The Ramadan comparison covers 3 seasons (90 days), so it shows a direction rather than a precise forecast.

---

# 7. Returns

**What we see**
- 1,056 returns worth **EGP 4.01M**, equal to 8.8% of sales value and 8.8% of orders.
- 77.6% of returns were refunded, 14.2% are still pending and 8.2% were rejected.

| Return Reason | Returned Value | Share of Value | Count |
|---|---|---|---|
| Duplicate Order | EGP 857,429 | 21.4% | 197 |
| Changed Mind | EGP 739,373 | 18.4% | 172 |
| Quality Issue | EGP 652,840 | 16.3% | 158 |
| Wrong Item Delivered | EGP 627,005 | 15.6% | 175 |
| Size Issue | EGP 617,976 | 15.4% | 173 |
| Defective Product | EGP 518,491 | 12.9% | 181 |

**What it means**
- **Duplicate orders and wrong items delivered together make up 37% of returned value (about EGP 1.48M).** These come from checkout and fulfilment errors, not from the products, and the business can fix them directly.
- Quality and defect issues (29% combined) point to supplier management, and size issues (15%) point to sizing guidance for clothing.
- 14% of returns are still pending, which suggests a processing backlog.

---

# Recommendations

| # | Recommendation | Based on | Expected impact |
|---|---|---|---|
| 1 | **Grow clothing:** give it more floor space, stock depth and marketing, especially in Small stores | Clothing = 54% of profit at a 47% margin | Higher group margin without needing more traffic |
| 2 | **Bundle electronics with higher-margin add-ons** (accessories, warranties, audio) | Electronics margin of 13% | Raises margin on the largest revenue stream |
| 3 | **Add duplicate-order checks at the point of sale and verify items at fulfilment** | 37% of returned value is preventable | Up to ~EGP 0.5M a year in avoided returns |
| 4 | **Rebuild the loyalty tiers around actual spend** and target the top 20% of customers | Top 20% = 59% of revenue; tiers show no difference in spend | Better retention of the highest-value customers |
| 5 | **Promote clothing during Ramadan and move electronics campaigns to the weeks around it** | Electronics −19% during Ramadan, clothing +3% | Smaller seasonal dip |
| 6 | **Grow footfall at Small stores** (local marketing, weekend events) | Small stores are short on order volume, not basket value | Less dependence on Greater Cairo |
| 7 | **Make employee ID a required field at checkout** | 5.3% of orders have no employee | Reliable staff performance and commission reporting |
| 8 | **Clear the backlog of pending returns** | 14% of returns are pending | Better customer experience, more accurate refund reporting |

---

# Limitations and assumptions

- **Total Sales is gross sales.** It includes orders marked *Cancelled* (EGP 1.68M) and *Returned*. Returns are reported separately in the Returns measures rather than subtracted from sales.
- **Returns are linked to the date dimension only.** In the current model, returns can be analysed by date but not directly by store, product or customer.
- **Employee attribution reflects who processed an order**, not necessarily who made the sale.
- **Loyalty tier rules are unknown**, so the finding that tiers don't track spend is based on observed spend only.
- **Ramadan analysis is based on 3 seasons** and should be treated as directional.
- **The dataset is fictional and educational.** The findings show the analytical method, not facts about a real business.
