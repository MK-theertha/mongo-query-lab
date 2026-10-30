# MongoDB Interview Practice — Question Bank

Use database `interview_practice` and collections `customers`, `products`, `orders`, `reviews`, and `employees`. Import each JSON file as a **JSON array** in MongoDB Compass. Dates use MongoDB Extended JSON (`$date`) and should import as BSON dates.

Try solving the questions in Compass's **Documents** filter first, then use **Aggregations** for pipeline questions. Unless a question says otherwise, don't change the source data.

## A. CRUD and query fundamentals
1. Find all active customers.
2. Find customers in Dubai or Abu Dhabi.
3. Find products priced between 100 and 300 inclusive.
4. Find products that are active and have stock greater than 0.
5. Find products with `tags` containing `wireless`.
6. Find products with either `Electronics` or `Fitness` category.
7. Find orders with status `delivered` and total above 300.
8. Find orders placed in a date range in 2025.
9. Find customers whose loyalty tier is not `bronze`.
10. Find products where stock is 0 or the product is inactive.
11. Return only product name, category, price; exclude `_id`.
12. Sort products by price descending and return the 5 most expensive.
13. Find orders using `card` payment and sort newest first.
14. Find reviews with rating 4 or 5.
15. Explain the difference between `find()` and `findOne()`; `updateOne()` and `updateMany()`.

## B. Updates, deletes, and operators
16. Increase stock by 10 for product `P001` using `$inc`.
17. Set `P003` stock to 25 and update its active flag.
18. Add a new tag to `P001` without duplicating existing tags (`$addToSet`).
19. Remove the `couponCode` field from orders where it is null (think carefully about matching null vs missing).
20. Mark all inactive customers as active (then revert or reset the dataset).
21. Use an upsert to create or update a product by `_id`.
22. Explain `$set`, `$unset`, `$inc`, `$push`, `$addToSet`, `$pull`, and `upsert`.
23. What is the difference between `deleteOne()` and `deleteMany()`? How would you avoid accidental mass updates/deletes?

## C. Aggregation fundamentals
24. Count orders by status using `$group` and `$sum`.
25. Calculate total revenue from delivered orders only.
26. Calculate average order value by status.
27. Count customers by country and sort highest count first.
28. Calculate average, minimum, and maximum product price per category.
29. Find categories with at least 3 products using `$match` after `$group`.
30. Calculate each product's gross margin amount (`price - cost`) and margin percentage with `$addFields` or `$project`.
31. Group orders by payment method and calculate order count, revenue, and average total.
32. Find the top 5 customers by number of orders.
33. Find the top 5 customers by total spend; decide whether cancelled/returned orders should count and state your assumption.
34. Calculate monthly order count and revenue using `$dateTrunc` or `$year` + `$month`.
35. Calculate revenue by country using `shippingAddress.country`.
36. Use `$unwind` to calculate quantity sold per product from `orders.items`.
37. Calculate revenue by item category by unwinding `items`.
38. Find orders with at least 3 items (array length, not quantity sum).
39. Find the average rating and review count for each product.
40. Find products whose average review rating is below 4.
41. Find each customer's first and most recent order using `$min` and `$max`.
42. Use `$facet` to return multiple summaries in one aggregation: status counts, payment-method totals, and top products by units sold.
43. Use `$bucket` or `$bucketAuto` to group products into price bands.
44. Use `$cond` or `$switch` to label products as `low`, `mid`, or `premium` price.
45. Explain the difference between `$match`, `$project`, `$set`/`$addFields`, `$group`, `$sort`, `$limit`, `$unwind`, and `$count`.

## D. `$lookup`, joins, and nested data
46. Join orders to customers with `$lookup`, showing customer name and order total.
47. Join reviews to products and show product name with each rating.
48. Join reviews to customers and show reviewer names.
49. Join products to reviews and include products with no reviews (left-outer join behavior).
50. Use `$lookup` with a pipeline to find orders for each customer, then calculate order count and spend.
51. Join employees to their managers using a self-lookup on `managerId`.
52. Join orders to products by unwinding `items` and matching `items.productId` to products `_id`; compare this with the order item's embedded snapshot.
53. Explain when to embed versus reference documents, and the trade-offs of duplicating product name/price inside an order item.

## E. Advanced aggregation and edge cases
54. Find the most frequently purchased products by units, not just number of orders.
55. Find customers who have never placed an order (`$lookup` + empty array or `$setDifference`).
56. Find products that have never been reviewed.
57. Calculate each customer's spend, order count, and average order value in a single pipeline.
58. Use `$reduce` to calculate the total quantity in each order without `$unwind`.
59. Use `$map` to transform each order item into `{ productId, lineTotal }`.
60. Use `$filter` to keep only order items whose line total exceeds 200.
61. Use `$arrayElemAt` or `$first` to extract an element from an array safely.
62. Handle missing, null, and empty arrays using `$ifNull`, `$type`, `$isArray`, and `$cond`.
63. Use `$regex` for a case-insensitive product-name search; explain why regex patterns can affect index use.
64. Use `$setWindowFields` to rank products by price within each category (MongoDB version support matters).
65. Use `$densify` or `$fill` conceptually for time-series gaps; explain when useful.
66. Use `$merge` to write a computed product summary into a separate collection (only after understanding that it changes data).
67. Explain `$out` versus `$merge`.
68. Explain when `$facet` can be useful and its memory/output limitations.
69. Explain why `$unwind` can change document counts and how `preserveNullAndEmptyArrays` works.
70. Design an aggregation that returns a paginated order list plus a total count using `$facet`.

## F. Indexes, performance, and production judgement
71. Create an index supporting product category + price filtering. Explain compound index field order.
72. Create an index for orders filtered by `customerId` and sorted by `orderDate` descending.
73. Use `explain('executionStats')` to compare a query before and after indexing.
74. Explain `COLLSCAN`, `IXSCAN`, `nReturned`, `totalDocsExamined`, and `totalKeysExamined`.
75. When would a multikey index be created, and what limitations can arrays introduce?
76. Explain unique, sparse, partial, TTL, and compound indexes; which would suit customer email?
77. Why can a leading-wildcard regex be slow?
78. How would you diagnose a slow aggregation pipeline and where should `$match` and `$project` go?
79. What are the 16 MB BSON document limit and the risks of unbounded arrays?
80. Explain read concern, write concern, transactions, and replica sets at a high level.
81. How would you design pagination for a very large orders collection: `skip/limit` versus range-based pagination?
82. How would you prevent duplicate orders if an API request is retried? Discuss unique keys/idempotency.
83. What is the difference between aggregation and MapReduce, and when should aggregation be preferred?
84. Explain sharding, shard keys, and why a poor shard key causes hotspots or scatter-gather queries.

## G. Schema design and interview scenarios
85. Design collections for customers, products, orders, and reviews. Which fields would you embed and which reference?
86. A dashboard needs daily revenue, top products, and order counts. Would you calculate live or maintain a summary collection?
87. An order has thousands of items or a customer accumulates millions of events. How would you prevent unbounded document growth?
88. A product price changes after an order is placed. How should historical order totals remain correct?
89. An aggregation gives duplicate totals after `$lookup`. What might have multiplied the rows and how would you fix it?
90. A query is fast locally but slow in production. What would you inspect: indexes, data volume, explain plans, working set, network, or pipeline shape?
91. A field can be absent, null, or an empty array. How can each affect `$match`, `$unwind`, and `$size`?
92. A requirement says “top-selling product.” Clarify whether it means number of orders, units sold, or revenue.
93. A collection has a high write rate and many indexes. Explain the read/write trade-off of indexes.
94. A report must include customers with zero orders. Outline a `$lookup`-based solution.
95. A collection has inconsistent types (string prices and numeric prices). How would you detect and migrate the data safely?

## Suggested practice order
- First: 1–23 (CRUD and operators)
- Then: 24–45 (aggregation)
- Then: 46–70 (`$lookup`, arrays, advanced stages)
- Finally: 71–95 (performance, schema design, production scenarios)

**Interview habit:** For every aggregation, say what each stage does, what shape the documents have after that stage, and whether the metric means count, units, or revenue. State assumptions about cancelled/returned orders.
