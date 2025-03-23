# Inbank_Najmin_Internship
# Task 2 – Currency-Normalized Transaction Aggregation

## 📌 Objective

The goal of this task is to **return total transaction amounts in euros**, aggregated by `transaction_date`, based on the given data from four related tables:

- `payments`
- `currencies`
- `currency_rates`
- `blacklist`

## ✅ Requirements

To generate the final result, the query must:

1. **Convert amounts into EUR**  
   - Use `currency_rates` to convert all non-EUR payments to EUR using the applicable exchange rate at the transaction date.

2. **Exclude discontinued currencies**  
   - Any payment made in a currency where `currencies.end_date IS NOT NULL` should be ignored (e.g., HRK, YUM).

3. **Exclude blacklisted users**  
   - If a user appears in the `blacklist` table with an active blacklist period covering the `transaction_date`, their payments must be excluded.

4. **Aggregate**  
   - Group by `transaction_date` and return the **SUM in EUR** per date.

---

## 📂 Input Tables

- **`payments`** – Contains raw transaction data including user, amount, currency, and transaction date.
- **`currencies`** – Metadata for currency codes, including start and end dates (used for validation).
- **`currency_rates`** – Exchange rate history with `EXCHANGE_DATE`, used to convert amounts into EUR.
- **`blacklist`** – Tracks users who are blacklisted during certain time periods.

---

## 🧠 Logic Summary

1. **Filter out discontinued currencies**  
   Join `payments` with `currencies` and exclude rows where `currencies.end_date IS NOT NULL`.

2. **Join the latest applicable exchange rate**  
   Use `currency_id` and match on `exchange_date = transaction_date`.

3. **Convert amounts to EUR**  
   Use exchange rate only if currency is not EUR. If already EUR, no conversion is needed.

4. **Remove blacklisted users**  
   Exclude any user whose `blacklist_start_date <= transaction_date` and 
   (`blacklist_end_date` IS NULL or `blacklist_end_date > transaction_date`).

5. **Aggregate**  
   Final result: `SUM(amount_eur)` grouped by `transaction_date`.

---

## 🛠️ SQL Tools & Notes

- Query is written in standard SQL and was tested on **SQL Server**.
- Used **CASE** logic for conditional conversion.
- Used **LEFT JOIN** to filter blacklisted users and apply data quality rules.

---

## 📈 Output

| transaction_date | sum_amount_eur |
|------------------|----------------|
| 2023-01-05       | 115.82         |
| 2023-02-05       | 100.20         |
| 2023-03-05       | 318.00         |

---

## 🔎 Notes & Edge Cases

- **Currency HRK** is excluded as it was discontinued on `2022-12-31`.
- **Blacklisted user 3837** is excluded from all transactions since their blacklist has no end date.
- **Currency rates are assumed to be daily**, matched 1:1 with `transaction_date`.

---

## 🧩 Next Steps / Improvements

- Add fallback logic if exchange rate is missing (e.g. latest available).
- Add monitoring on currency data quality (completeness, timeliness, null rate).
- Integrate with dbt tests or Great Expectations for pipeline validation in production.

