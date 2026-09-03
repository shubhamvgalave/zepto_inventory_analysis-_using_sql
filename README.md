# 🛒 Zepto Inventory Data Analysis using SQL

## 📌 Project Overview

This project focuses on analyzing **Zepto e-commerce inventory data using MySQL**.

The objective of this project is to simulate a real-world data analyst workflow, starting from a raw inventory dataset and using SQL to:

* Explore the structure and quality of the data
* Identify missing and inconsistent values
* Clean and transform the dataset
* Analyze product pricing and discounts
* Understand stock availability
* Estimate potential revenue
* Identify value-for-money products
* Analyze inventory weight across categories

The project was performed using **MySQL Workbench**, with the dataset imported directly from a CSV file.

---

## 📊 Dataset Overview

The dataset contains Zepto product inventory information. Each row represents a product/SKU available in the inventory.

### Main Columns

| Column                   | Description                                              |
| ------------------------ | -------------------------------------------------------- |
| `name`                   | Name of the product                                      |
| `category`               | Product category                                         |
| `mrp`                    | Maximum Retail Price                                     |
| `discountPercent`        | Discount percentage                                      |
| `discountedSellingPrice` | Selling price after discount                             |
| `availableQuantity`      | Available inventory quantity                             |
| `weightInGms`            | Product weight in grams                                  |
| `outOfStock`             | Indicates whether the product is out of stock            |
| `quantity`               | Number of units/package                                  |
| `sku_id`                 | Unique auto-incremented identifier added during analysis |

---

## 🛠️ Tools & Technologies

* **MySQL**
* **MySQL Workbench**
* **SQL**
* **CSV Dataset**

---

## 🔄 Project Workflow

### 1. 📥 Data Import

The raw Zepto CSV dataset was imported **directly into MySQL Workbench**.

Instead of manually creating the complete table structure beforehand, the CSV data was imported into MySQL first.

After importing the dataset, a unique `sku_id` column was added using:

```sql
ALTER TABLE zepto
ADD COLUMN sku_id INT AUTO_INCREMENT PRIMARY KEY;
```

This created a unique identifier for each product record.

---

### 2. 🔍 Data Exploration

The first step was to understand the dataset and identify potential data-quality issues.

#### Total Number of Records

```sql
SELECT COUNT(*) FROM zepto;
```

#### View Sample Records

```sql
SELECT *
FROM zepto
LIMIT 10;
```

#### Check for NULL Values

The dataset was checked for missing values across important columns such as:

* Product name
* Category
* MRP
* Discount
* Selling price
* Weight
* Available quantity
* Stock status
* Quantity

```sql
SELECT *
FROM zepto
WHERE name IS NULL
OR category IS NULL
OR mrp IS NULL
OR discountPercent IS NULL
OR discountedSellingPrice IS NULL
OR weightInGms IS NULL
OR availableQuantity IS NULL
OR outOfStock IS NULL
OR quantity IS NULL;
```

#### Identify Product Categories

```sql
SELECT DISTINCT category
FROM zepto
ORDER BY category;
```

#### Compare In-Stock and Out-of-Stock Products

```sql
SELECT outOfStock, COUNT(sku_id)
FROM zepto
GROUP BY outOfStock;
```

#### Find Products Having Multiple SKUs

The same product name can appear multiple times because different SKUs may represent different package sizes or product variants.

```sql
SELECT name, COUNT(sku_id) AS "Number of SKUs"
FROM zepto
GROUP BY name
HAVING COUNT(sku_id) > 1
ORDER BY COUNT(sku_id) DESC;
```

---

## 🧹 3. Data Cleaning

After exploring the dataset, data-cleaning operations were performed.

### Identify Products with Zero Price

```sql
SELECT *
FROM zepto
WHERE mrp = 0
OR discountedSellingPrice = 0;
```

### Remove Invalid MRP Records

```sql
DELETE FROM zepto
WHERE mrp = 0;
```

### Convert Paise to Rupees

The original price values were stored in **paise**, so the MRP and discounted selling price were converted into **Indian Rupees (₹)**.

```sql
UPDATE zepto
SET mrp = mrp / 100.0,
    discountedSellingPrice = discountedSellingPrice / 100.0;
```

The converted prices were then verified:

```sql
SELECT mrp, discountedSellingPrice
FROM zepto;
```

---

# 📈 4. Business Analysis

After cleaning the data, SQL queries were used to answer practical business questions.

---

## Q1. What are the top 10 best-value products based on discount percentage?

```sql
SELECT DISTINCT name, mrp, discountPercent
FROM zepto
ORDER BY discountPercent DESC
LIMIT 10;
```

**Purpose:**
Identify products offering the highest percentage discounts.

---

## Q2. Which products have a high MRP but are out of stock?

```sql
SELECT DISTINCT name, mrp
FROM zepto
WHERE outOfStock = 'TRUE'
AND mrp > 300
ORDER BY mrp DESC;
```

**Purpose:**
Identify expensive products that are currently unavailable, which can help highlight potential inventory or replenishment opportunities.

---

## Q3. What is the estimated revenue for each category?

```sql
SELECT category,
       SUM(discountedSellingPrice * availableQuantity) AS total_revenue
FROM zepto
GROUP BY category
ORDER BY total_revenue;
```

**Purpose:**
Estimate the potential revenue represented by the available inventory in each category.

> **Note:** This is an estimated inventory-value calculation, not actual historical sales revenue.

---

## Q4. Which products have an MRP greater than ₹500 and a discount below 10%?

```sql
SELECT DISTINCT name, mrp, discountPercent
FROM zepto
WHERE mrp > 500
AND discountPercent < 10
ORDER BY mrp DESC, discountPercent DESC;
```

**Purpose:**
Identify relatively expensive products that have limited discounts.

---

## Q5. Which 5 categories offer the highest average discount?

```sql
SELECT category,
       ROUND(AVG(discountPercent), 2) AS avg_discount
FROM zepto
GROUP BY category
ORDER BY avg_discount DESC
LIMIT 5;
```

**Purpose:**
Compare discount strategies across product categories.

---

## Q6. Which products provide the best price per gram?

Products weighing at least 100 grams were analyzed.

```sql
SELECT DISTINCT name,
       weightInGms,
       discountedSellingPrice,
       ROUND(discountedSellingPrice / weightInGms, 2) AS price_per_gram
FROM zepto
WHERE weightInGms >= 100
ORDER BY price_per_gram;
```

**Purpose:**
Identify products that provide better value based on their price relative to weight.

---

## Q7. How can products be grouped based on their weight?

Products were classified into three categories:

* **Low:** Less than 1000g
* **Medium:** 1000g to less than 5000g
* **Bulk:** 5000g or more

```sql
SELECT DISTINCT name,
       weightInGms,
       CASE
           WHEN weightInGms < 1000 THEN 'Low'
           WHEN weightInGms < 5000 THEN 'Medium'
           ELSE 'Bulk'
       END AS weight_category
FROM zepto;
```

**Purpose:**
Create meaningful weight-based product groups for inventory analysis.

---

## Q8. What is the total inventory weight for each category?

```sql
SELECT category,
       SUM(weightInGms * availableQuantity) AS total_weight
FROM zepto
GROUP BY category
ORDER BY total_weight;
```

**Purpose:**
Measure the total physical inventory weight held within each product category.

---

# 💡 Key SQL Concepts Used

This project demonstrates practical use of:

* `SELECT`
* `WHERE`
* `DISTINCT`
* `COUNT()`
* `SUM()`
* `AVG()`
* `ROUND()`
* `GROUP BY`
* `HAVING`
* `ORDER BY`
* `LIMIT`
* `CASE`
* `ALTER TABLE`
* `DELETE`
* `UPDATE`
* `AUTO_INCREMENT`
* Primary Keys
* Data Cleaning
* Business-oriented SQL analysis

---

# 📁 Project Structure

```text
zepto-sql-data-analysis/
│
├── zepto_analysis.sql
├── zepto_v2.csv
└── README.md
```

> `zepto_analysis.sql` contains the SQL queries used for data exploration, cleaning, and business analysis.

---

# 🎯 Project Objectives

The main objectives of this project were to:

1. Import and work with a real-world e-commerce dataset.
2. Explore the dataset using SQL.
3. Identify missing and invalid data.
4. Add a unique SKU identifier.
5. Clean pricing-related data.
6. Convert paise into rupees.
7. Analyze discounts and product pricing.
8. Analyze stock availability.
9. Estimate inventory-based revenue.
10. Analyze inventory weight and product value.

---

# 🚀 What I Learned

Through this project, I gained practical experience in using SQL for **real-world data analysis**.

I learned how to move beyond basic SQL queries and use SQL to:

* Investigate messy datasets
* Perform data cleaning
* Create useful derived classifications
* Analyze inventory
* Answer business questions
* Generate insights from e-commerce data

This project helped me understand how SQL can be used as a **data analysis and business intelligence tool**, rather than just a database querying language.

---

## 👨‍💻 Author

**Shubham Galave**

*Bachelor of Information Technology Engineering Student*

---

⭐ If you found this project useful, consider giving the repository a star!
