
# 📚 Online Bookstore – SQL Data Analysis

A PostgreSQL project that models an online bookstore with three related tables (**Books**, **Customers**, **Orders**) and answers 20 business questions using SQL — from basic filtering to joins, aggregations and inventory calculations.

---

## 📁 Repository Structure

```
.
├── Books.csv                 # 500 books (catalogue + stock)
├── Customers.csv             # 500 customers
├── Orders.csv                # 500 orders
├── sql project query.sql     # Table creation + all analysis queries
└── README.md
```

---

## 🗄️ Database Schema

```
Customers (1) ────< Orders >──── (1) Books
```

| Table | Columns |
|-------|---------|
| **Books** | `Book_ID` (PK), `Title`, `Author`, `Genre`, `Published_Year`, `Price`, `Stock` |
| **Customers** | `Customer_ID` (PK), `Name`, `Email`, `Phone`, `City`, `Country` |
| **Orders** | `Order_ID` (PK), `Customer_ID` (FK → Customers), `Book_ID` (FK → Books), `Order_Date`, `Quantity`, `Total_Amount` |

---

## 📊 Dataset Overview

| Metric | Value |
|--------|-------|
| Books / Customers / Orders | 500 / 500 / 500 |
| Genres | 7 (Science Fiction, Mystery, Romance, Fantasy, Non-Fiction, Biography, Fiction) |
| Book price range | $5.07 – $49.98 |
| Publication years | 1900 – 2023 |
| Total units in stock | 25,056 |
| Order date range | Dec 2022 – Dec 2024 |
| Total revenue | **$75,628.66** |
| Countries / Cities represented | 215 / 489 |
| Distinct customers who ordered | 307 |
| Distinct books ordered | 317 |
| Missing values | None in any table |

> The data is synthetic (randomly generated names, titles and locations), so it is well suited for practising SQL rather than drawing real-world business conclusions.

---

## ⚙️ Setup

1. Create a PostgreSQL database (e.g. `OnlineBookstore`).
2. Run the `CREATE TABLE` section of `sql project query.sql`.
3. Import the CSVs (order matters because of foreign keys):

```sql
COPY Books(Book_ID, Title, Author, Genre, Published_Year, Price, Stock)
FROM '/path/to/Books.csv' CSV HEADER;

COPY Customers(Customer_ID, Name, Email, Phone, City, Country)
FROM '/path/to/Customers.csv' CSV HEADER;

COPY Orders(Order_ID, Customer_ID, Book_ID, Order_Date, Quantity, Total_Amount)
FROM '/path/to/Orders.csv' CSV HEADER;
```

(Or use pgAdmin's *Import/Export Data* wizard.)

4. Run the queries in the same file.

---

## 🔍 Questions Answered

### Basic Queries
| # | Question | Result on this dataset |
|---|----------|------------------------|
| 1 | Books in the "Fiction" genre | 60 books |
| 2 | Books published after 1950 | 292 books |
| 3 | Customers from Canada | 3 customers |
| 4 | Orders placed in November 2023 | 25 orders |
| 5 | Total stock available | 25,056 units |
| 6 | Most expensive book | *Proactive system-worthy orchestration* (Robert Scott, Mystery) – $49.98 |
| 7 | Orders with quantity > 1 | 438 orders |
| 8 | Orders with total amount > $20 | 473 orders |
| 9 | All genres | 7 genres |
| 10 | Book with the lowest stock | 5 books have 0 stock (query returns just one) |
| 11 | Total revenue | $75,628.66 |

### Advanced Queries
| # | Question | Concepts used |
|---|----------|---------------|
| 1 | Total books sold per genre | `JOIN`, `SUM`, `GROUP BY` |
| 2 | Average price of Fantasy books (≈ $25.98) | `AVG` |
| 3 | Customers with multiple orders | `GROUP BY`, `HAVING` |
| 4 | Most frequently ordered book | `JOIN`, `COUNT`, `ORDER BY … LIMIT` |
| 5 | Top 3 most expensive Fantasy books | `ORDER BY`, `LIMIT` |
| 6 | Total quantity sold per author | `JOIN`, `SUM` |
| 7 | Cities of customers who spent over $30 | `JOIN`, `WHERE` |
| 8 | Customer who spent the most (Kim Turner – $1,398.90) | `JOIN`, `SUM`, `LIMIT` |
| 9 | Stock remaining after fulfilling all orders | `LEFT JOIN`, `COALESCE` |

---

## 💡 Key Insights

- **Mystery** is the best-selling genre by units (504), followed by Science Fiction (447), Fantasy (446) and Romance (439). **Biography** (285) and **Fiction** (225) sell least.
- **Patrick Contreras** is the top-selling author with 28 units sold.
- **Kim Turner** is the highest-spending customer ($1,398.90), followed by Jonathon Strickland ($1,080.95) and Carrie Perez ($1,052.27).
- Most customers are one-time buyers: 168 of 307 purchasing customers placed a single order; 139 placed two or more.
- Orders are heavily weighted to 2023 (256) and 2024 (228), with only 16 in late 2022.
- **Inventory risk:** 5 books are already at 0 stock, and after fulfilling all orders **30 books would have negative stock** (up to 18 units oversold), which points to a stock-tracking gap that should be investigated.
- `Total_Amount` equals `Quantity × Price` for every order, so the order data is internally consistent.

---

## 🛠️ Notes & Possible Improvements to the SQL

A few queries behave slightly differently from their comments. Suggested fixes:

1. **Basic Q4 (November 2023):** `BETWEEN '2023-11-01' AND '2023-11-30'` works for a `DATE` column, but `order_date >= '2023-11-01' AND order_date < '2023-12-01'` is safer if the column ever becomes a timestamp.
2. **Basic Q7:** the comment says "customers", but the query returns orders. To list customers, join to `Customers` and use `DISTINCT`.
3. **Basic Q10:** `LIMIT 1` hides ties (5 books have 0 stock). Use `WHERE stock = (SELECT MIN(stock) FROM books)` to return all of them.
4. **Advanced Q3:** the comment says *at least 2 orders*, but `HAVING COUNT(order_id) > 2` returns customers with **3 or more** (41 customers). It should be `>= 2` (139 customers).
5. **Advanced Q4:** 7 books tie for the most orders (4 each); `LIMIT 1` returns only one arbitrarily. Add a tie-breaker or use `RANK()`.
6. **Advanced Q7:** the result lists the same city multiple times and does not total each customer's spend. Use `GROUP BY c.customer_id, c.city HAVING SUM(o.total_amount) > 30` with `DISTINCT` cities if "customers who spent over $30" means total spend.
7. **Advanced Q9:** the query groups by `b.book_id` only; this works in PostgreSQL (primary key functional dependency) but listing `b.title` and `b.stock` in `GROUP BY` is more portable.
8. **Typos:** `stock_aviable` → `stock_available`; the heading `--9)` is missing a space.

---

## 🚀 Skills Demonstrated

- Relational schema design with primary and foreign keys
- Filtering, sorting and limiting (`WHERE`, `ORDER BY`, `LIMIT`)
- Aggregations (`SUM`, `AVG`, `COUNT`, `DISTINCT`)
- Joins (`INNER JOIN`, `LEFT JOIN`) and `GROUP BY` / `HAVING`
- Null handling with `COALESCE`
- Data validation and business-logic checking

---

## 🧰 Tech Stack

- **PostgreSQL** (uses `SERIAL`, `NUMERIC`)
- pgAdmin / psql (any PostgreSQL client)

---

## 👤 Author

*Saurav, www.linkedin.com/in/saurav-kashyap-21794b351.*
