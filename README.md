# mongo-query-lab

A small, synthetic MongoDB dataset with practice questions, worked answers, and a quick-reference guide for preparing for MongoDB interviews.

## Contents
- `PRACTICE_QUESTIONS.md` — the questions, grouped by topic (CRUD, aggregation, joins, schema design, …)
- `answers.md` — worked `mongosh` solutions to every question (try the questions first)
- `field-guide.html` — a standalone MongoDB / mongosh reference; open it in any browser
- `*.json` — the five collections described below

## Collections
- `customers.json` — 12 customers, location, status, loyalty tier
- `products.json` — 15 products, category, prices/costs, stock, tags
- `orders.json` — 50 orders, nested `items` arrays, dates, status, discounts, shipping, payments
- `reviews.json` — 12 reviews referencing products and customers
- `employees.json` — 7 employees with `managerId` for self-lookup practice

IDs are strings to make references easy to inspect in Compass. Dates use Extended JSON `$date` and should import as BSON dates.

## Import into MongoDB Compass
1. Connect to your local MongoDB deployment.
2. Create database `interview_practice` (or let Compass create it during collection import).
3. Create a collection for each file: `customers`, `products`, `orders`, `reviews`, `employees`.
4. Open each collection → **Add Data** → **Import JSON or CSV** → select the matching `.json` file.
5. Choose JSON and import. Each file contains a JSON array of documents.
6. Open `PRACTICE_QUESTIONS.md` and solve in order. Check your work against `answers.md`.

Prefer the command line? From this folder, run one per collection:
```
mongoimport --db interview_practice --collection customers --jsonArray --file customers.json
```

## Notes
- This is synthetic practice data, not real customer data.
- Orders intentionally have mixed statuses, discounts, payment methods, and multiple line items.
- Order item names/prices are snapshots. Use `products` for current catalogue values and discuss the difference in schema-design questions.
- Some customers/products have no matching orders/reviews so you can practise left-join and “never purchased/reviewed” questions.
