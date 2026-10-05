# Olist Pricing Strategy & Voucher Effectiveness

**Data Sprint, Group 2**: Ayman, Isa, Alfadhul

## Problem Statement

Olist, a Brazilian e-commerce platform, asked a consulting team whether its pricing team is making the right decisions, given that customer retention is low. We analysed about 99,000 orders (late 2016 to October 2018) to find out whether vouchers and installment payments raise order value, sales and loyalty, and where they do not.

## Executive Summary

**Approach.** We worked with the Olist Brazilian E-Commerce Public Dataset (orders, payments, order items, products, customers and reviews). Three analysts each took a set of the client's questions and worked in their own notebook; the three are combined into one notebook in this repo (`Code/olist_data_sprint_combined.ipynb`). The data was joined at order level, and customers were identified with `customer_unique_id`, because `customer_id` changes with every order and would make every customer look like a first-time buyer. We did not use the marketing funnel dataset, because it covers seller acquisition and has no link to pricing, vouchers or installments.

**Findings.** Installments matter. Orders paid in more than one installment are 51.5% of orders but 63.5% of sales (R$10.2M of R$16.0M), and their mean order value is 64% higher (R$199 vs R$121). Order value rises with the number of installments, from R$121 for one installment to R$360 for more than ten, and the average number of installments rises with order value up to about R$600, then levels off at around seven. Use of installments differs by category: the ten categories with the highest installment share are paid in installments for 64% to 85% of their sales (`la_cuisine` and `pcs` lead), and `pcs` has the highest average installment count (seven). The ten states with the highest installment share sit in a narrow band of 61% to 68%. Vouchers are small and do not look like a growth lever: voucher orders are 3.9% of orders and 3.3% of sales, and their mean order value is 16% lower (R$136 vs R$162). Credit cards make up 76% of single-method orders and carry the highest order values, while voucher-only orders are the smallest (median R$72 vs R$109 for credit cards). In 2017, the only full year in the data, voucher payments peak in November (387) and July (364), and there is no August peak. Vouchers also do not bring in new customers: 93.8% of voucher orders are from first-time buyers, against 96.8% of other orders, and 96.9% of all customers ordered only once. Freight averages 30.8% of the product price (20.9% of the product plus freight total), and cancellations are rare.

**Recommendations.** Keep installments and credit card checkout as the priority. Do not count on vouchers to raise order value, attract new customers or build loyalty, and test them with a controlled experiment before spending more. The bigger opportunity is retention, since nearly all customers never come back.

## Client Questions and Where They Are Answered

| Client question | Status | Notebook section |
|---|---|---|
| Are discounts, installments and vouchers effective? | Answered (vouchers used as the closest proxy for discounts) | Part 1, Q1 |
| Do vouchers increase sales or harm margins? | Sales answered; margin not measurable (no cost or voucher-funding data) | Part 1, Q2 |
| Are installment payments helping or hurting? | Answered | Part 1, Q3 |
| Why is there a voucher peak in August? | Answered: no August peak in 2017 | Part 2 |
| How does payment type relate to order value? | Answered | Part 2 |
| Are vouchers attracting new first-time buyers? | Answered | Part 2 |
| Is a category very expensive or often bought with installments? | Answered | Part 3 |
| Are installments related to category or location? | Answered | Part 3 |
| Are vouchers distributed effectively (by state)? | Answered | Part 3 |
| What is the % for cheaper vs expensive items? | Answered | Part 3 |
| Is the freight value reasonable? | Partly: freight averages 30.8% of product price, with no benchmark to judge it against | Part 3 |

## File Directory

```
.
├── README.md
├── Data/                                  # Olist CSV files, unmodified
├── Code/
│   └── olist_data_sprint_combined.ipynb   # all three parts, run top to bottom
└── Presentation/
    └── Data_Sprint_G2.pptx                # slides (add the PDF export here too)
```

The notebook is organised as **Setup**, then **Part 1** (installments and vouchers, Alfadhul), **Part 2** (voucher timing, payment types and first-time buyers, Ayman) and **Part 3** (products, shipping, installments and cancellations, Isa). Part 1 also saves chart images (`1.png`, `2.png`, `3.png`, `g.png`, `ff.png`, `x.png`) next to the notebook.

**To reproduce:** put the Olist CSV files in `Data/`, open the notebook from `Code/`, and run **Restart and Run All**. It needs Python 3 with `pandas`, `numpy` and `matplotlib`. All paths are relative (`../Data/`).

## Data and Data Dictionary

**Source.** [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) (Kaggle): about 100,000 orders placed on Olist marketplaces between 2016 and 2018, with payments, items, products, customers and reviews. Only 2017 is a full calendar year; 2016 covers the last months only, and 2018 stops in October with very few orders after August.

**Columns used from the original files:**

| File | Columns used | Meaning |
|---|---|---|
| `olist_orders_dataset` | `order_id`, `customer_id`, `order_status`, `order_purchase_timestamp` | Order, its customer key, status (for example `delivered`, `canceled`) and purchase time |
| `olist_order_payments_dataset` | `payment_sequential`, `payment_type`, `payment_installments`, `payment_value` | One row per payment. `payment_type` is `credit_card`, `boleto`, `voucher`, `debit_card` or `not_defined` |
| `olist_order_items_dataset` | `order_item_id`, `product_id`, `seller_id`, `shipping_limit_date`, `price`, `freight_value` | One row per item in an order |
| `olist_products_dataset` | `product_id`, `product_category_name` | Product category |
| `olist_customers_dataset` | `customer_id`, `customer_unique_id`, `customer_state` | `customer_id` is new for every order; `customer_unique_id` identifies the person |
| `olist_order_reviews_dataset` | `order_id` and review fields | Merged into the Part 3 order table but not used in any chart |

The sellers, geolocation and category-translation files are loaded in the Setup section but are not used in the analysis.

**Engineered features:**

| Feature | Part | Definition |
|---|---|---|
| `used_voucher` (order level) | 1, 2 | True if at least one payment row of the order is a `voucher` |
| `used_installments` | 1, 3 | True if the order was paid in more than one installment |
| `installment_group` | 1 | Installment count binned as 1, 2 to 3, 4 to 6, 7 to 10, and more than 10 |
| `total_orders`, `total_spend`, `avg_order_value` | 1 | Customer-level totals per `customer_unique_id` |
| `month`, `month_name`, `year` | 2 | Taken from `order_purchase_timestamp` |
| `voucher_payments`, `all_payments`, `voucher_share_pct` | 2 | Weekly counts and voucher share, for the weeks around Black Friday 2017 |
| `order_totals` | 2 | Order value for orders paid with one payment type only |
| `order_number`, `is_first_order`, `n_orders`, `came_back` | 2 | Order sequence per customer, first-order flag, orders per customer and whether a first-time buyer ordered again |
| `price_category` | 3 | "More Expensive" if the category's mean price is above the mean across categories, otherwise "Cheaper" |
| `total_price`, `total_freight`, `full_price` | 3 | Sum of item prices, sum of freight, and their total per order |
| `payment_range` | 3 | `full_price` in bands of R$100 up to R$2,000, then more than R$2,000 |

**Definitions to keep in mind.**
- A *voucher order* in Part 1 and in the first-time-buyer analysis has at least one voucher payment. Part 2's payment-type comparison uses *voucher-only* orders. This is why voucher shares differ between sections (3.9% of orders vs 1.7%).
- The monthly and weekly voucher charts count payment rows, not orders.
- *First-time buyer* means the customer's first order in this dataset.
- Part 3 uses inner merges, so an order missing from any of its tables is dropped and counts can be slightly lower than in Part 1.

## Conclusions and Recommendations

**Conclusions**
- Installments are central to Olist sales: 51.5% of orders and 63.5% of sales, with higher order values as the number of installments grows. Their use varies by category (up to about 85% of a category's sales) and is high across states (61% to 68% in the top ten).
- Vouchers play a small role (3.9% of orders, 3.3% of sales) and are linked to lower order values, not higher. Voucher use is concentrated in a few states: São Paulo accounts for about 41% of voucher use, then Rio de Janeiro (about 14%) and Minas Gerais (about 11%).
- Credit cards are 76% of single-method orders and have the highest order value; voucher-only orders have the lowest.
- There is no August voucher peak in 2017. November (387) and July (364) are the highest months. The week of Black Friday (24 November 2017) has the most voucher payments (142), and the voucher share of all payments that week (4.5%) is in line with the other November weeks (4.6% to 6.1%).
- Vouchers are not attracting new customers: 6.2% of voucher orders come from returning customers, against 3.2% of other orders. Of first-time buyers who used a voucher, 3.9% ordered again, against 3.1% of the rest (5.1% vs 4.2% in 2017 alone). The gap is small.
- Retention is the main problem: 96.9% of customers placed only one order.
- Cancellations are rare. No state has more than about 2.2% of its orders cancelled (Roraima), most are below 0.7%, and installment orders are only slightly more likely to be cancelled than single-payment orders.
- Margins cannot be measured with this data. As an illustration only, at the brief's assumed 10 to 15% take-rate, voucher orders' sales of about R$0.52M would earn Olist roughly R$52,000 to R$79,000 in commission.

**Recommendations**
1. **Keep installments available.** They accompany larger orders and nearly two-thirds of sales. This is an association, not proof that installments cause larger baskets.
2. **Do not rely on vouchers** to raise order value, bring in new customers or build loyalty. The data shows no clear benefit on any of the three.
3. **Test vouchers before scaling them.** Offer them to a random group of customers and compare order value, repeat purchases and margin against a group without them. Find out who funds each voucher.
4. **Keep credit card checkout the priority.**
5. **Focus on retention.** With about 97% of customers buying once, test levers other than vouchers, such as follow-up messages or loyalty offers.
6. **Keep monitoring cancellations.** They are rare, and installment orders are only slightly more affected.

## Areas for Further Research

- Get voucher campaign dates and who pays for each voucher, so that margins and the November and July peaks can be explained.
- Use several years of data to separate seasonal patterns from platform growth.
- Add external demographic and regional data (for example from the Brazilian census) to answer the regional price and demographic questions.
- Compare the same product across sellers to see whether voucher use differs.
- Compare freight with a benchmark (carrier rates, or competitor shipping fees) before judging whether it is reasonable.
- Review the marketing funnel dataset for seller acquisition, which was out of scope here.
- Run a controlled voucher experiment, as recommended above.

## Limitations

- Only 2017 is a full year; 2016 and 2018 are partial, so seasonal findings rest on a single year.
- The dataset has no discount field, no product cost, and no information on who funds vouchers, so vouchers stand in for discounts and margins cannot be calculated.
- Results show associations, not causes. For example, people who use vouchers may already buy cheaper items.
- Calendar explanations for the November and July peaks (Black Friday, Father's Day) show matching timing only.
- Some payment rows have type `not_defined` (value 0); these are excluded from the payment-type comparison.
- "First-time buyer" is limited to the period in this dataset.

## Sources

- Olist, [Brazilian E-Commerce Public Dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce), Kaggle.
- Olist, [Marketing Funnel dataset](https://www.kaggle.com/datasets/olistbr/marketing-funnel-olist), Kaggle (reviewed, not used).
- Forbes Brasil (August 2017), [Vendas da Black Friday no varejo online devem subir até 20%](https://forbes.com.br/negocios/2017/08/vendas-da-black-friday-no-varejo-online-devem-subir-ate-20/): date of Black Friday 2017 (24 November).
- Monitor Mercantil, [Vendas virtuais devem crescer 10% no Dia dos Pais](https://monitormercantil.com.br/vendas-virtuais-devem-crescer-10-no-dia-dos-pais/): Father's Day 2017 (13 August) as the first important date for online retail in the second half of the year.

## Key Visualizations

| Chart | Notebook section | Message |
|---|---|---|
| Mean order value by number of installments | Part 1, Q1 | Order value rises with the number of installments |
| Total sales: installments vs single payment | Part 1, Q3 | Installment orders carry 63.5% of sales |
| Voucher payments by month, 2017 | Part 2 | No August peak; November and July are highest |
| Voucher payments per week, November 2017 | Part 2 | Black Friday week has the most voucher payments |
| Share of orders and order value by payment type | Part 2 | Credit cards dominate; voucher-only orders are smallest |
| First-time vs returning customers, by voucher use | Part 2 | Voucher orders are no more likely to be from new customers |
| Customers who ordered once vs again | Part 2 | About 97% of customers never return |
| Installment use by category and by state | Part 3 | Installment use is high everywhere and highest in a few categories |
| Average installments by price range | Part 3 | More installments for pricier orders, levelling off at around seven |
| Distribution of vouchers among states | Part 3 | São Paulo accounts for about 41% of voucher use |
| Cancelled orders as a share of each state's orders | Part 3 | Cancellations are rare in nearly every state |
