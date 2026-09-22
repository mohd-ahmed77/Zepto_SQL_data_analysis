# 🛒 Zepto Inventory & Pricing Analytics

**SQL-powered business analysis of a quick-commerce product catalog**

`SQL` · `PostgreSQL` · `Data Analytics`

> Turning 3,731 rows of raw product data into simple, useful business insights — using nothing but SQL.

---

## 📌 Overview

Zepto is a quick-commerce app (order online, get it in minutes). Behind every product sits a small record: its price, discount, stock status, category, and weight. This project asks a set of plain business questions of that catalog and answers them with SQL — filtering, sorting, grouping, aggregation, and conditional logic — then explains each finding in plain English.

The goal is twofold:

- **For anyone:** understand *what we asked, what we found, and why it matters*.
- **For a technical reviewer:** see real SQL applied to real business questions.

---

## 🗂️ Dataset

| | |
|---|---|
| **Source** | Zepto's public product listings, scraped (via Kaggle) |
| **Type** | Catalog / inventory snapshot — **not** transactions |
| **Raw rows** | 3,732 |
| **After cleaning** | 3,731 |
| **Categories** | 14 |
| **Unique product names** | 1,680 (products repeat as different pack sizes/SKUs) |

**Columns:** `name`, `category`, `mrp`, `discountPercent`, `discountedSellingPrice`, `availableQuantity`, `weightInGms`, `outOfStock`, `quantity` (package qty).

### Cleaning steps
- Removed **1 row** with `MRP = 0` (*Cherry Blossom Liquid Shoe Polish*).
- Converted prices from **paise → rupees** (÷100) for `mrp` and `discountedSellingPrice`.

---

## 🧰 Tech Stack

- **PostgreSQL** — database & query engine
- **SQL** — `SELECT`, `WHERE`, `ORDER BY`, `LIMIT`, `GROUP BY`, `SUM`, `AVG`, `ROUND`, `CASE WHEN`
- **CSV** — raw data import (`zepto_v2.csv`)

---

## ❓ Business Questions & Key Findings

### 1. Which products have the biggest discounts?
```sql
SELECT DISTINCT name, mrp, discountPercent
FROM zepto
ORDER BY discountPercent DESC
LIMIT 10;
```
**Finding:** The highest discount reaches **51%** (*Dukes Waffy Wafers*). Top discounts cluster in **cheese, yogurt, and wafer** products.

### 2. Which expensive items are out of stock?
```sql
SELECT DISTINCT name, mrp
FROM zepto
WHERE outOfStock = TRUE AND mrp > 300
ORDER BY mrp DESC;
```
**Finding:** **453** items are out of stock overall (~12%). Only **4** are priced above ₹300 — a short restock watchlist:

| Product | MRP |
|---|---|
| Patanjali Cow's Ghee | ₹565 |
| MamyPoko Pants Standard Diapers, XL | ₹399 |
| Aashirvaad Atta with Multigrains | ₹315 |
| Everest Kashmiri Lal Chilli Powder | ₹310 |

### 3. Where is inventory value concentrated?
```sql
SELECT category,
       SUM(discountedSellingPrice * availableQuantity) AS est_value
FROM zepto
GROUP BY category
ORDER BY est_value DESC;
```
**Finding:** **Cooking Essentials** and **Munchies** lead at ≈ **₹3.37 lakh** each.
> ⚠️ This is the *estimated value of available stock* (selling price × quantity) — **not** actual sales.

### 4. Which items cost a lot but are barely discounted?
```sql
SELECT DISTINCT name, mrp, discountPercent
FROM zepto
WHERE mrp > 500 AND discountPercent < 10
ORDER BY mrp DESC;
```
**Finding:** **39 products** match — dominated by **large cooking-oil jars** (premium price, little discount).

### 5. Which categories discount the most?
```sql
SELECT category,
       ROUND(AVG(discountPercent), 2) AS avg_discount
FROM zepto
GROUP BY category
ORDER BY avg_discount DESC
LIMIT 5;
```
**Finding:** **Fruits & Vegetables** discount most at **15.46%** — roughly double the catalog-wide average of **7.62%**.

### 6. Which products give the best value per gram?
```sql
SELECT DISTINCT name, weightInGms, discountedSellingPrice,
       ROUND(discountedSellingPrice / weightInGms, 2) AS price_per_gram
FROM zepto
WHERE weightInGms >= 100
ORDER BY price_per_gram;
```
**Finding:** Staples and produce win — **Onion, Tata Salt, Aashirvaad Salt** at ≈ **₹0.02/g**.

### 7. Small, medium, or bulk packs?
```sql
SELECT
  CASE
    WHEN weightInGms < 1000 THEN 'Low'
    WHEN weightInGms < 5000 THEN 'Medium'
    ELSE 'Bulk'
  END AS weight_bucket,
  COUNT(*) AS products
FROM zepto
GROUP BY weight_bucket;
```
**Finding:** The catalog is overwhelmingly small packs — **Low: 3,392 · Medium: 293 · Bulk: 46**.

### 8. How much does the inventory weigh?
```sql
SELECT category,
       SUM(weightInGms * availableQuantity) AS total_weight
FROM zepto
GROUP BY category
ORDER BY total_weight DESC;
```
**Finding:** ≈ **6 tonnes** (6,005.8 kg) total, led by Cooking Essentials & Munchies (≈ 1,405 kg each).

---

## 📊 Summary of Insights

| Theme | Takeaway |
|---|---|
| 💰 Pricing | Discounts reach 51%, but the average is only **7.6%** |
| 📦 Stock | **453** items out of stock (~12%) |
| 🏷️ Categories | Fruits & Veg discount most (**15.5%**) |
| ⚖️ Value | Price-per-gram compares pack sizes fairly |
| 📊 Inventory | ≈ **₹22.4 lakh** estimated value, ≈ **6 tonnes** |

---

## 🚀 How to Run

```bash
# 1. Create the database and table in PostgreSQL
psql -U your_user -d your_db -f schema.sql

# 2. Load the dataset
\copy zepto FROM 'zepto_v2.csv' DELIMITER ',' CSV HEADER;

# 3. Run the analysis queries
psql -U your_user -d your_db -f analysis.sql
```

*(Adjust file names to match your repo.)*

---

## ⚠️ Limitations

This analysis is based on **public, scraped catalog/inventory data** — not transactions. It does **not** reveal:

- Real orders or actual sales
- Profit or margins
- Customer behavior
- Zepto's internal decisions

**Estimated inventory value ≠ actual sales.** Naming these limits is part of responsible analysis.

---

## 👤 Author

**MOHD AHMED** — Data Scientist
`SQL` · `Data Analytics` · `Business Intelligence`

- LinkedIn: https://www.linkedin.com/in/mohammad-ahmed-094859245/

- Portfolio: https://mohdahmed.netlify.app/

---

> *Good analytics isn't just writing queries — it's asking the right questions and explaining the answers clearly.*
