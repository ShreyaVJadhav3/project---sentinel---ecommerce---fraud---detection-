# 🛡️ Project Sentinel : E-Commerce Fraud Detection & Data Leak Analysis

![Python](https://img.shields.io/badge/Python-3.x-blue) ![SQL](https://img.shields.io/badge/SQL-MS%20SQL%20Server-red) ![PowerBI](https://img.shields.io/badge/PowerBI-Dashboard-yellow) ![Dataset](https://img.shields.io/badge/Dataset-Olist%20Brazil-green)

---

## 📌 Project Overview

**Project Sentinel** is an end-to-end data analysis pipeline built to detect fraudulent order patterns — specifically "Ghost Orders" — in e-commerce transaction data. A Ghost Order is defined as an order where a product was successfully delivered to the customer but the payment value was near zero or completely bypassed, representing a direct financial loss to the platform.

This project was inspired by real-world consumer complaints about fraudulent e-commerce transactions observed across social media platforms in India. The Brazilian Olist e-commerce dataset was used as a proxy to model and investigate similar fraud patterns at scale.

---

## 🗂️ Project Pipeline

```
Raw CSV Data (Olist Kaggle Dataset)
        ↓
MS SQL Server — Schema Design + Fraud Detection Queries
        ↓
Python (Pandas + NumPy + Matplotlib + Seaborn) — Data Cleaning + EDA + Feature Engineering
        ↓
Power BI — 2-Page Interactive Dashboard
        ↓
Key Findings + Business Insights
```

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| **MS SQL Server (SSMS)** | Database design, fraud pattern queries |
| **Python (Pandas, NumPy)** | Data cleaning, feature engineering, EDA |
| **Matplotlib & Seaborn** | Python visualizations |
| **Power BI Desktop** | Interactive 2-page dashboard |
| **Kaggle - Olist Dataset** | Brazilian e-commerce transaction data (99,441 orders) |

---

## 📊 Dataset

**Source:** [Brazilian E-Commerce Public Dataset by Olist — Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

The dataset contains 8 interconnected tables:

| Table | Records | Description |
|---|---|---|
| orders | 99,441 | Order status and timestamps |
| order_payments | 1,03,886 | Payment type and value |
| order_items | 1,12,650 | Product, seller, price per item |
| customers | 99,441 | Customer location data |
| products | 32,951 | Product category and dimensions |
| sellers | 3,095 | Seller location data |
| order_reviews | 99,224 | Review scores and comments |
| product_category_name_translation | 71 | Portuguese to English category mapping |

---

## 🔍 Phase 1 — SQL Analysis (MS SQL Server)

Four fraud detection queries were written to identify suspicious patterns:

**Query 1 — Identity Theft Detection**
Flags customers placing orders from multiple geographic locations using `COUNT(DISTINCT zip_code_prefix)` — a signal for account compromise or identity fraud.

**Query 2 — Ghost Delivery Detection**
Identifies orders with `order_status = 'delivered'` but `order_approved_at IS NULL` — meaning products were shipped without financial gateway approval. A follow-up query cross-references payment type to address the Cash on Delivery exception.

**Query 3 — Value Leakage Analysis**
Detects orders where `payment_value < (price * 0.8)` — flagging transactions where customers paid less than 80% of the product price, indicating payment bypass fraud.

**Query 4 — Master Fraud Query**
Combines identity theft signals and value leakage into a single query across 4 joined tables to rank the highest risk customers by `suspected_revenue_loss`.

---

## 🐍 Phase 2 — Python EDA & Feature Engineering

### Data Cleaning
- **products_df** — 610 null category names filled using Portuguese word `'outro'` (other), then merged with English translation table via left join
- **Physical dimensions** — 2 null values filled using **median** (outlier-resistant strategy)
- **orders_df** — Date columns converted from object to `datetime64` using a loop
- **order_reviews_df** — 87,656 null comment titles and 58,247 null messages filled with placeholder text

### Feature Engineering
Two original features were engineered:

**`process_gap`** — A categorical column created using `np.select()` to classify null patterns in orders:
- `Critical_Bypass` — Delivered but never approved (14 orders)
- `Approval_Leak` — Not approved but in non-delivered status (146 orders)
- `In_Progress` — Simply not yet delivered (2,819 orders)
- `Standard_Process` — Normal orders (96,462 orders)

**`is_customer_silent`** — A binary flag (1/0) created using `np.where()` to identify ghost order victims who never left a review comment, enabling silent victim analysis.

### Master Table
All 8 tables merged into a single analytical master dataframe (119,143 rows) using sequential left joins on `order_id` and `product_id`. Revenue leak was calculated using aggregated order-level pricing to avoid item-level duplication errors.

### Ghost Order Detection
Ghost Orders were defined as transactions where:
```python
payment_value == 0 OR payment_value < (price * 0.1)
```
After deduplication and null removal: **586 unique Ghost Orders detected**

---

## 📈 Phase 3 — Power BI Dashboard (2 Pages)

### Page 1 — E-Commerce Data Leak Overview

| Visual | Insight |
|---|---|
| KPI: 586 Ghost Orders | Scale of fraud detected |
| KPI: R$ 80.86K Revenue Lost | Total financial damage |
| KPI: R$ 137.98 Avg Leak Per Order | Average cost per fraud incident |
| KPI: 61.26% Silent Rate | Majority of victims never complained |
| Breach Heartbeat Timeline | Fraud persisted across 22 months (Oct 2016 — Aug 2018) |
| Top 10 Victim Categories | bed_bath_table leads with 80 ghost orders |
| Revenue Leak by Category & Payment Type | Stacked bar showing credit card vs voucher exploitation |
| Payment Method Exploitation Donut | 61.77% credit card, 38.23% voucher |
| Did the Victims Speak Up? | 61.26% silent, 38.74% complained |

### Page 2 — Project Sentinel : Full Platform Analysis

| Visual | Insight |
|---|---|
| KPI: 99K Total Orders | Full platform scale |
| KPI: 0.59% Fraud Rate | Industry-realistic fraud rate |
| KPI: R$ 18M Total Revenue | Platform revenue context |
| KPI: 0.45% Revenue Loss | Fraud as % of total revenue |
| Platform Order Growth Trend | Steady growth pattern 2016-2018 |
| Top 10 Categories by Total Revenue | bed_bath_table and health_beauty dominate |
| Product Wise Average Payment Value | Computers highest at R$ 1,269 |
| Payment Type Distribution | 75% credit card platform-wide vs 62% in ghost orders |

---

## 🔑 Key Findings

1. **586 Ghost Orders** were detected across 22 months, causing **R$ 80,860 in direct revenue loss**

2. **Fraud Rate of 0.59%** — within industry standard range (0.5%-2%), but the financial impact is disproportionate

3. **Voucher Exploitation** — Vouchers represent only 4% of all platform payments but account for 38.23% of ghost order payments — a **9x overrepresentation** indicating systematic voucher abuse

4. **Mid-Value Category Targeting** — Fraudsters concentrated on mid-value categories like `bed_bath_table` (R$ avg ~180) rather than high-value categories like `computers` (R$ 1,269 avg), deliberately avoiding premium payment verification checks

5. **Silent Victim Pattern** — 61.26% of ghost order victims never left a complaint review, making this fraud largely invisible to platform monitoring systems

6. **Fraud Disguised as Normal Transactions** — The majority of ghost orders were classified as `Standard_Process`, meaning they bypassed fraud detection by mimicking legitimate order behavior

7. **health_beauty** is both a top revenue category (R$ 1.7M) AND a top fraud victim category — making it a **critical business priority** for fraud prevention

---

## 📁 Repository Structure

```
Project-Sentinel/
│
├── SQL/
│   └── SQL_Query.sql               # All 4 fraud detection queries
│
├── Python/
│   └── EDA_Notebook.ipynb          # Full data cleaning + EDA notebook
│
├── Dashboard/
│   ├── Page1_Ghost_Order_Audit.png # Page 1 screenshot
│   ├── Page2_Full_Platform.png     # Page 2 screenshot
│   └── Final_Dashboard.pdf         # Full dashboard PDF export
│
├── Data/
│   └── Database_Schema.pdf         # Hand-drawn database schema diagram
│
└── README.md
```

---

## 🚀 How to Reproduce

1. Download the Olist dataset from [Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
2. Load all CSV files into MS SQL Server using SSMS
3. Run the queries from `SQL/SQL_Query.sql`
4. Open `Python/EDA_Notebook.ipynb` and run all cells sequentially
5. Import `Sentinel_Ghost_Orders.csv` and `Sentinel_Master_Data.csv` into Power BI Desktop
6. Recreate the dashboard using the exported CSVs

---

## 👩‍💻 Author

**Shreya Jadhav**
GitHub: [@ShreyaVJadhav3](https://github.com/ShreyaVJadhav3)

---

## 📌 Acknowledgements

Dataset: [Olist Brazilian E-Commerce Dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) — made publicly available on Kaggle
