https://claude.ai/artifact/6MNwsWmx4uEMidT1dJAL1c 

============================================================
A. CRUD AND QUERY FUNDAMENTALS
============================================================

1. Find all active customers. 
db.customers.find({ status: "active" })

2. Find customers in Dubai or Abu Dhabi.
db.customers.find({ city: { $in: ["Dubai", "Abu Dhabi"] } })

3. Find products priced between 100 and 300 inclusive.
db.products.find({ price: { $gte: 100, $lte: 300 } })

4. Find active products with stock greater than zero.
db.products.find({ active: true, stock: { $gt: 0 } })

5. Find products whose tags array contains "wireless".
db.products.find({ tags: "wireless" })
MongoDB matches an array field when one element equals the specified value.

6. Find products in either Electronics or Fitness.
db.products.find({ category: { $in: ["Electronics", "Fitness"] } })

7. Find delivered orders with total above 300.
db.orders.find({ status: "delivered", total: { $gt: 300 } })

8. Find orders placed during 2025.‼️
db.orders.find({
  orderDate: {
    $gte: ISODate("2025-01-01T00:00:00Z"),
    $lt: ISODate("2026-01-01T00:00:00Z")
  }
})
A half-open interval avoids end-of-day precision problems.

9. Find customers whose loyalty tier is not bronze.
db.customers.find({ loyaltyTier: { $ne: "bronze" } })
Note: $ne can also match documents where the field is missing. Add $exists:true if needed.

10. Find products where stock is zero OR the product is inactive.
db.products.find({ $or: [{ stock: 0 }, { active: false }] })

11. Return only product name, category, price; exclude _id.
db.products.find({}, { _id: 0, name: 1, category: 1, price: 1 })

12. Sort products by price descending and return five.
db.products.find().sort({ price: -1 }).limit(5)

13. Find card-payment orders, newest first.
db.orders.find({ paymentMethod: "card" }).sort({ orderDate: -1 })

14. Find reviews with rating 4 or 5.
db.reviews.find({ rating: { $in: [4, 5] } })

15. Explain find/findOne and updateOne/updateMany.
***********************************************************************
- find() returns a cursor containing zero or more matching documents.
- findOne() returns one matching document or null.
- updateOne() modifies at most one matching document.
- updateMany() modifies every matching document.
- insertOne() inserts one document; insertMany() inserts multiple documents.
- replaceOne() replaces a document's contents (subject to _id rules); updateOne() changes fields using update operators.
- upsert:true inserts a document if no document matches the filter.
***********************************************************************

============================================================
B. UPDATES, DELETES, AND OPERATORS
============================================================

16. Increase stock by 10 for P001 using $inc.
db.products.updateOne({ _id: "P001" }, { $inc: { stock: 10 } })

17. Set P003 stock to 25 and mark it active.
db.products.updateOne(
  { _id: "P003" },
  { $set: { stock: 25, active: true } }
)

18. Add a tag without duplicates using $addToSet.
db.products.updateOne(
  { _id: "P001" },
  { $addToSet: { tags: "ergonomic" } }
)
🔥$push always appends; $addToSet avoids adding the same value again.

19. Remove couponCode when it is explicitly null.
db.orders.updateMany(
  { $and: [
      { couponCode: null },
      { couponCode: { $exists: true } }
  ] },
  { $unset: { couponCode: "" } }
)
{couponCode:null} alone matches both explicit null and missing fields. The $exists condition limits this to fields that exist.

20. Mark all inactive customers active (then reset the dataset).
db.customers.updateMany(
  { status: "inactive" },
  { $set: { status: "active" } }
)

21. Upsert a product by _id.
db.products.updateOne(
  { _id: "P016" },
  { $set: {
      name: "USB Cable", category: "Electronics", price: 19.5,
      cost: 5, stock: 100, tags: ["computer"], active: true, rating: 4
  }},
  { upsert: true }
)

22. Explain common update operators.
***********************************************************************
- $set: set or replace a field's value.
- $unset: remove a field.
- $inc: increment/decrement a numeric value.
- $push: append an array element, allowing duplicates.
- $addToSet: append only if the value is not already present.
- $pull: remove array elements matching a condition.
- upsert: an update option that inserts a new document if no document matches.
Example: db.products.updateOne({_id:"P001"}, {$pull:{tags:"wireless"}})


23. deleteOne vs deleteMany; avoiding accidental mass changes.
deleteOne() deletes at most one matching document; deleteMany() deletes every matching document. First test the filter with find() and countDocuments(). Use specific filters, backups, least-privilege credentials, and transactions where appropriate. Avoid an empty filter {} unless removing all documents is intentional.
***********************************************************************

============================================================
C. AGGREGATION FUNDAMENTALS
============================================================

24. Count orders by status.
db.orders.aggregate([
  { $group: { _id: "$status", orderCount: { $sum: 1 } } },
  { $sort: { orderCount: -1 } }
])

25. Total revenue from delivered orders only.
db.orders.aggregate([
  { $match: { status: "delivered" } },
  { $group: {
      _id: null, revenue: { $sum: "$total" }, orderCount: { $sum: 1 }
  } }
])
Clarify whether revenue includes shipping/tax and how refunds/discounts are treated.

26. Average order value by status.
db.orders.aggregate([
  { $group: {
      _id: "$status",
      averageOrderValue: { $avg: "$total" },
      orderCount: { $sum: 1 }
  }},
  { $sort: { averageOrderValue: -1 } }
])

27. Count customers by country, highest count first.
db.customers.aggregate([
  { $group: { _id: "$country", customerCount: { $sum: 1 } } },
  { $sort: { customerCount: -1 } }
])

28. Average, minimum, maximum product price per category.
db.products.aggregate([
  { $group: {
      _id: "$category",
      averagePrice: { $avg: "$price" },
      minimumPrice: { $min: "$price" },
      maximumPrice: { $max: "$price" },
      productCount: { $sum: 1 }
  }},
  { $sort: { _id: 1 } }
])

29. Categories with at least three products.
db.products.aggregate([
  { $group: { _id: "$category", productCount: { $sum: 1 } } },
  { $match: { productCount: { $gte: 3 } } },
  { $sort: { productCount: -1 } }
])

30. Calculate gross margin amount and margin percentage.
db.products.aggregate([
  { $project: {
      name: 1, price: 1, cost: 1,
      margin: { $subtract: ["$price", "$cost"] },
      marginPercent: {
        $cond: [
          { $gt: ["$price", 0] },
          { $multiply: [
              { $divide: [{ $subtract: ["$price", "$cost"] }, "$price"] },
              100
          ]},
          null
        ]
      }
  }}
])
Margin percentage = (price - cost) / price * 100. Markup percentage instead divides by cost.

31. Order count, revenue, and average total by payment method.
db.orders.aggregate([
  { $group: {
      _id: "$paymentMethod",
      orderCount: { $sum: 1 },
      revenue: { $sum: "$total" },
      averageTotal: { $avg: "$total" }
  }},
  { $sort: { revenue: -1 } }
])

32. Top five customers by number of orders.
db.orders.aggregate([
  { $group: { _id: "$customerId", orderCount: { $sum: 1 } } },
  { $sort: { orderCount: -1 } },
  { $limit: 5 },
  { $lookup: {
      from: "customers", localField: "_id",
      foreignField: "_id", as: "customer"
  }},
  { $unwind: "$customer" },
  { $project: {
      _id: 0, customerId: "$_id",
      name: "$customer.name", orderCount: 1
  }}
])

33. Top five customers by spend, excluding cancelled and returned orders.
db.orders.aggregate([
  { $match: { status: { $nin: ["cancelled", "returned"] } } },
  { $group: {
      _id: "$customerId",
      spend: { $sum: "$total" },
      orderCount: { $sum: 1 }
  }},
  { $sort: { spend: -1 } },
  { $limit: 5 },
  { $lookup: {
      from: "customers", localField: "_id",
      foreignField: "_id", as: "customer"
  }},
  { $unwind: "$customer" },
  { $project: {
      _id: 0, customerId: "$_id", name: "$customer.name",
      spend: 1, orderCount: 1
  }}
])
This defines spend as total for statuses other than cancelled/returned. State the rule in an interview.

34. Monthly order count and revenue.
db.orders.aggregate([
  { $match: { status: { $nin: ["cancelled", "returned"] } } },
  { $group: {
      _id: { $dateTrunc: { date: "$orderDate", unit: "month" } },
      orderCount: { $sum: 1 },
      revenue: { $sum: "$total" },
      averageOrderValue: { $avg: "$total" }
  }},
  { $sort: { _id: 1 } }
])
$dateTrunc is available in modern MongoDB versions. Alternative: group by {$year:"$orderDate"} and {$month:"$orderDate"}.

35. Revenue by shipping country.
db.orders.aggregate([
  { $match: { status: { $nin: ["cancelled", "returned"] } } },
  { $group: {
      _id: "$shippingAddress.country",
      revenue: { $sum: "$total" },
      orderCount: { $sum: 1 }
  }},
  { $sort: { revenue: -1 } }
])

36. Quantity sold per product using $unwind.
db.orders.aggregate([
  { $match: { status: { $nin: ["cancelled", "returned"] } } },
  { $unwind: "$items" },
  { $group: {
      _id: "$items.productId",
      productName: { $first: "$items.name" },
      unitsSold: { $sum: "$items.quantity" }
  }},
  { $sort: { unitsSold: -1 } }
])

37. Revenue by item category.
db.orders.aggregate([
  { $match: { status: { $nin: ["cancelled", "returned"] } } },
  { $unwind: "$items" },
  { $group: {
      _id: "$items.category",
      lineRevenue: {
        $sum: { $multiply: ["$items.unitPrice", "$items.quantity"] }
      },
      unitsSold: { $sum: "$items.quantity" }
  }},
  { $sort: { lineRevenue: -1 } }
])
This calculates line-item revenue before order-level discount/shipping allocation.

38. Orders with at least three line items (not total quantity).
db.orders.find({ $expr: { $gte: [{ $size: "$items" }, 3] } })
If items may be missing or non-array, guard with $isArray and $cond.

39. Average rating and review count per product.
db.reviews.aggregate([
  { $group: {
      _id: "$productId",
      averageRating: { $avg: "$rating" },
      reviewCount: { $sum: 1 }
  }},
  { $lookup: {
      from: "products", localField: "_id",
      foreignField: "_id", as: "product"
  }},
  { $unwind: "$product" },
  { $project: {
      _id: 0, productId: "$_id", name: "$product.name",
      averageRating: 1, reviewCount: 1
  }},
  { $sort: { averageRating: -1 } }
])

40. Products with average rating below 4.
db.reviews.aggregate([
  { $group: {
      _id: "$productId",
      averageRating: { $avg: "$rating" },
      reviewCount: { $sum: 1 }
  }},
  { $match: { averageRating: { $lt: 4 } } },
  { $lookup: {
      from: "products", localField: "_id",
      foreignField: "_id", as: "product"
  }},
  { $unwind: "$product" },
  { $project: {
      _id: 0, name: "$product.name", averageRating: 1, reviewCount: 1
  }}
])

41. First and latest order date per customer.
db.orders.aggregate([
  { $group: {
      _id: "$customerId",
      firstOrder: { $min: "$orderDate" },
      latestOrder: { $max: "$orderDate" },
      orderCount: { $sum: 1 }
  }},
  { $sort: { _id: 1 } }
])

42. Use $facet for status counts, payment totals, and top products by units.
db.orders.aggregate([
  { $facet: {
      statusCounts: [
        { $group: { _id: "$status", count: { $sum: 1 } } },
        { $sort: { count: -1 } }
      ],
      paymentTotals: [
        { $group: { _id: "$paymentMethod", total: { $sum: "$total" } } },
        { $sort: { total: -1 } }
      ],
      topProducts: [
        { $match: { status: { $nin: ["cancelled", "returned"] } } },
        { $unwind: "$items" },
        { $group: {
            _id: "$items.productId",
            name: { $first: "$items.name" },
            units: { $sum: "$items.quantity" }
        }},
        { $sort: { units: -1 } },
        { $limit: 5 }
      ]
  }}
])
Each facet processes the same input stream and returns its own result array.

43. Group products into price bands with $bucket.
db.products.aggregate([
  { $bucket: {
      groupBy: "$price",
      boundaries: [0, 50, 100, 200, 400, 700],
      default: "700+",
      output: { count: { $sum: 1 }, averagePrice: { $avg: "$price" } }
  }}
])

44. Label products low, mid, or premium.
db.products.aggregate([
  { $set: {
      priceBand: {
        $switch: {
          branches: [
            { case: { $lt: ["$price", 100] }, then: "low" },
            { case: { $lt: ["$price", 300] }, then: "mid" }
          ],
          default: "premium"
        }
      }
  }},
  { $project: { name: 1, price: 1, priceBand: 1 } }
])

45. Explain common aggregation stages.
***********************************************************************
- $match filters documents; place selective filters early when possible.
- $project selects, removes, or computes fields.
- $set/$addFields adds or replaces fields while retaining other fields.
- $group groups by a key and calculates aggregates.
- $sort sorts documents.
- $limit restricts the number of output documents.
- $unwind emits one document per array element.
- $count outputs a document containing the number of input documents.
***********************************************************************

============================================================
D. $LOOKUP, JOINS, AND NESTED DATA
============================================================

46. Join orders to customers and show customer name and order total.
db.orders.aggregate([
  { $lookup: {
      from: "customers", localField: "customerId",
      foreignField: "_id", as: "customer"
  }},
  { $unwind: "$customer" },
  { $project: {
      _id: 1, orderDate: 1, total: 1, status: 1,
      customerName: "$customer.name",
      customerEmail: "$customer.email"
  }}
])

47. Join reviews to products.
db.reviews.aggregate([
  { $lookup: {
      from: "products", localField: "productId",
      foreignField: "_id", as: "product"
  }},
  { $unwind: "$product" },
  { $project: {
      rating: 1, comment: 1, createdAt: 1,
      productName: "$product.name"
  }}
])

48. Join reviews to customers to show reviewer names.
db.reviews.aggregate([
  { $lookup: {
      from: "customers", localField: "customerId",
      foreignField: "_id", as: "customer"
  }},
  { $unwind: "$customer" },
  { $project: {
      rating: 1, comment: 1, reviewer: "$customer.name", createdAt: 1
  }}
])

49. Join products to reviews, including products with no reviews.
db.products.aggregate([
  { $lookup: {
      from: "reviews", localField: "_id",
      foreignField: "productId", as: "reviews"
  }},
  { $project: {
      name: 1, category: 1, reviews: 1,
      reviewCount: { $size: "$reviews" }
  }}
])
$lookup preserves products with no matches and gives them an empty array. If unwinding reviews, use preserveNullAndEmptyArrays:true to retain products without reviews.

50. Use a $lookup pipeline to calculate each customer's order count and spend.
db.customers.aggregate([
  { $lookup: {
      from: "orders",
      let: { customerKey: "$_id" },
      pipeline: [
        { $match: { $expr: { $eq: ["$customerId", "$$customerKey"] } } },
        { $match: { status: { $nin: ["cancelled", "returned"] } } },
        { $group: {
            _id: null, orderCount: { $sum: 1 }, spend: { $sum: "$total" }
        }}
      ],
      as: "orderSummary"
  }},
  { $set: {
      summary: {
        $ifNull: [
          { $arrayElemAt: ["$orderSummary", 0] },
          { orderCount: 0, spend: 0 }
        ]
      }
  }},
  { $project: {
      name: 1, orderCount: "$summary.orderCount", spend: "$summary.spend"
  }}
])

51. Join employees to their managers (self-lookup).
db.employees.aggregate([
  { $lookup: {
      from: "employees", localField: "managerId",
      foreignField: "_id", as: "manager"
  }},
  { $unwind: {
      path: "$manager", preserveNullAndEmptyArrays: true
  }},
  { $project: {
      name: 1, department: 1, title: 1, managerName: "$manager.name"
  }}
])

52. Join order items to current products and compare snapshots.
db.orders.aggregate([
  { $unwind: "$items" },
  { $lookup: {
      from: "products", localField: "items.productId",
      foreignField: "_id", as: "currentProduct"
  }},
  { $unwind: {
      path: "$currentProduct", preserveNullAndEmptyArrays: true
  }},
  { $project: {
      orderId: "$_id",
      orderedName: "$items.name",
      orderedPrice: "$items.unitPrice",
      currentName: "$currentProduct.name",
      currentPrice: "$currentProduct.price",
      quantity: "$items.quantity"
  }}
])
The embedded item is a snapshot of what was purchased; products holds current catalogue values.

53. When to embed versus reference.
Embed bounded data that is read together and belongs to its parent, such as order line-item snapshots. Reference shared, independently updated, many-to-many, or unbounded data, such as customers referenced by orders. Embedding can improve read locality but duplicates information and may grow documents. Referencing reduces duplication but may require extra queries or $lookup.

============================================================
E. ADVANCED AGGREGATION AND EDGE CASES
============================================================

54. Most purchased products by units, not order count.
db.orders.aggregate([
  { $match: { status: { $nin: ["cancelled", "returned"] } } },
  { $unwind: "$items" },
  { $group: {
      _id: "$items.productId",
      name: { $first: "$items.name" },
      unitsSold: { $sum: "$items.quantity" },
      lineCount: { $sum: 1 }
  }},
  { $sort: { unitsSold: -1 } },
  { $limit: 10 }
])
unitsSold is quantity; lineCount is how many order lines included the product.

55. Customers who have never placed an order.
db.customers.aggregate([
  { $lookup: {
      from: "orders", localField: "_id",
      foreignField: "customerId", as: "orders"
  }},
  { $match: { orders: { $eq: [] } } },
  { $project: { name: 1, email: 1, city: 1 } }
])

56. Products that have never been reviewed.
db.products.aggregate([
  { $lookup: {
      from: "reviews", localField: "_id",
      foreignField: "productId", as: "reviews"
  }},
  { $match: { reviews: { $eq: [] } } },
  { $project: { name: 1, category: 1 } }
])

57. Each customer's spend, order count, and average order value.
db.customers.aggregate([
  { $lookup: {
      from: "orders",
      let: { cid: "$_id" },
      pipeline: [
        { $match: { $expr: { $eq: ["$customerId", "$$cid"] } } },
        { $match: { status: { $nin: ["cancelled", "returned"] } } },
        { $group: {
            _id: null,
            spend: { $sum: "$total" },
            orderCount: { $sum: 1 },
            avgOrderValue: { $avg: "$total" }
        }}
      ],
      as: "stats"
  }},
  { $set: {
      stats: {
        $ifNull: [
          { $arrayElemAt: ["$stats", 0] },
          { spend: 0, orderCount: 0, avgOrderValue: null }
        ]
      }
  }},
  { $project: {
      name: 1, spend: "$stats.spend",
      orderCount: "$stats.orderCount",
      avgOrderValue: "$stats.avgOrderValue"
  }}
])

58. Calculate total item quantity per order using $reduce, without $unwind.
db.orders.aggregate([
  { $set: {
      totalUnits: {
        $reduce: {
          input: { $ifNull: ["$items", []] },
          initialValue: 0,
          in: { $add: ["$$value", "$$this.quantity"] }
        }
      }
  }},
  { $project: { items: 1, totalUnits: 1 } }
])

59. Use $map to produce {productId, lineTotal} for each item.
db.orders.aggregate([
  { $project: {
      itemTotals: {
        $map: {
          input: "$items",
          as: "item",
          in: {
            productId: "$$item.productId",
            lineTotal: { $multiply: ["$$item.unitPrice", "$$item.quantity"] }
          }
        }
      }
  }}
])

60. Use $filter to retain items whose line total exceeds 200.
db.orders.aggregate([
  { $project: {
      expensiveItems: {
        $filter: {
          input: "$items",
          as: "item",
          cond: {
            $gt: [
              { $multiply: ["$$item.unitPrice", "$$item.quantity"] },
              200
            ]
          }
        }
      }
  }}
])

61. Extract the first array element safely.
db.orders.aggregate([
  { $project: {
      firstItem: { $arrayElemAt: ["$items", 0] }
  }}
])
An empty array may yield no value for that field. Use $ifNull if a specific fallback is needed.

62. Handle missing, null, and empty arrays with $ifNull/$type/$isArray/$cond.
db.orders.aggregate([
  { $set: {
      safeItems: {
        $cond: [{ $isArray: "$items" }, "$items", []]
      }
  }},
  { $set: { itemCount: { $size: "$safeItems" } } }
])
$ifNull handles null/missing values but doesn't prove a value is an array. $type distinguishes BSON types, including "missing", "null", and "array".

63. Case-insensitive product-name search with regex.
db.products.find({ name: { $regex: "mouse", $options: "i" } })
An unanchored or case-insensitive regex may scan many documents. Anchored prefix searches can use a suitable index more effectively; check explain().

64. Rank products by price within each category using $setWindowFields.
db.products.aggregate([
  { $setWindowFields: {
      partitionBy: "$category",
      sortBy: { price: -1 },
      output: { priceRank: { $rank: {} } }
  }},
  { $project: { name: 1, category: 1, price: 1, priceRank: 1 } },
  { $sort: { category: 1, priceRank: 1 } }
])
Check the server version; window stages require a sufficiently recent MongoDB version.

65. Explain $densify and $fill.
After grouping time-series data by date, $densify can create missing date intervals and $fill can populate values according to a chosen rule. Example: add zero-sales days to a daily sales chart. Distinguish true zero from unknown/missing data and define date bounds/time zones.

66. Use $merge to write a product sales summary to another collection.
db.orders.aggregate([
  { $match: { status: { $nin: ["cancelled", "returned"] } } },
  { $unwind: "$items" },
  { $group: {
      _id: "$items.productId",
      unitsSold: { $sum: "$items.quantity" },
      lineRevenue: {
        $sum: { $multiply: ["$items.unitPrice", "$items.quantity"] }
      }
  }},
  { $merge: {
      into: "product_sales_summary",
      on: "_id",
      whenMatched: "replace",
      whenNotMatched: "insert"
  }}
])
$merge writes data. Test on a disposable database and decide how to remove stale summary documents.

67. Explain $out versus $merge.
$out writes the aggregation result to a collection, replacing that collection's contents. $merge supports matching existing documents and insert/replace/merge/keep behaviors, making it useful for materialized summaries. Both write data and have deployment/version/permission constraints.

68. When is $facet useful, and what are its limits?
$facet runs multiple independent sub-pipelines against the same input, e.g. totals, counts, and top results. Be aware of memory limits and the 16 MiB BSON document limit for the final output document. Apply shared selective $match stages before $facet when appropriate.

69. Why does $unwind change document counts?
$unwind emits one output document per array element, so an order with four items becomes four rows. By default, documents with missing, null, or empty arrays are dropped. preserveNullAndEmptyArrays:true keeps the parent document. Unwinding can multiply rows and accidentally inflate parent-level sums.

70. Paginated order list plus total count using $facet.
const page = 1;
const pageSize = 10;
db.orders.aggregate([
  { $match: { status: "delivered" } },
  { $sort: { orderDate: -1, _id: -1 } },
  { $facet: {
      data: [
        { $skip: (page - 1) * pageSize },
        { $limit: pageSize }
      ],
      metadata: [{ $count: "total" }]
  }}
])
If metadata is empty, total is zero. Deep $skip can be expensive; range pagination is often better for large datasets.

============================================================
F. INDEXES, PERFORMANCE, AND PRODUCTION
============================================================

71. Index for category + price filtering.
db.products.createIndex({ category: 1, price: 1 })
Compound index order matters. This index is suitable for category equality plus price range. Choose field order from actual filter/sort patterns and validate with explain().

72. Index orders by customerId and descending orderDate.
db.orders.createIndex({ customerId: 1, orderDate: -1 })
Supports equality filtering by customer and sorting that customer's orders newest first.

73. Compare explain("executionStats") before and after indexing.
db.orders.find({ customerId: "C001" })
  .sort({ orderDate: -1 })
  .explain("executionStats")
db.orders.createIndex({ customerId: 1, orderDate: -1 })
db.orders.find({ customerId: "C001" })
  .sort({ orderDate: -1 })
  .explain("executionStats")
Compare winningPlan, totalDocsExamined, totalKeysExamined, nReturned, and execution time. Small datasets may show little difference.

74. Explain execution plan terms.
- COLLSCAN: scans collection documents.
- IXSCAN: scans an index.
- nReturned: documents returned by a stage/plan.
- totalDocsExamined: documents examined.
- totalKeysExamined: index keys examined.
A high examined-to-returned ratio may indicate an inefficient query, but consider the query and plan context.

75. Multikey indexes and array limitations.
An index on an array field automatically becomes multikey, helping queries match array elements. Arrays can increase index size and write cost. Compound indexes have restrictions when multiple indexed fields are arrays in the same document. Verify restrictions for your MongoDB version and index design.

76. Unique, sparse, partial, TTL, and compound indexes.
- Unique: prevents duplicate indexed values.
- Sparse: indexes only documents where the indexed field exists.
- Partial: indexes only documents matching a filter.
- TTL: automatically expires documents after a configured time.
- Compound: indexes multiple fields in a defined order.
A unique customer-email index can be appropriate after duplicate cleanup. Decide how missing/null emails and case sensitivity should work; consider normalized emails or collation.

77. Why can leading-wildcard regex be slow?
A pattern like /.*mouse/ cannot usually use a normal B-tree index to narrow the scan efficiently. Prefer anchored prefix searches when appropriate, normalized searchable fields, or MongoDB Search for more advanced text search.

78. Diagnose a slow aggregation pipeline.
1. Run explain("executionStats") and inspect examined/returned counts.
2. Put selective $match stages early and index common filter/sort fields.
3. Avoid unnecessary $unwind and large intermediate results.
4. Reduce fields with $project when it meaningfully reduces work or transfer.
5. Inspect $lookup join keys, sort/group memory behavior, data cardinality, and server metrics.
6. Test with production-like data.
MongoDB can optimize pipelines internally; an early $project is not always faster by itself.

79. 16 MiB document limit and unbounded arrays.
A BSON document has a 16 MiB maximum size. Unbounded arrays can grow documents, slow updates, increase contention, and eventually exceed the limit. Use references, child collections, bucket patterns, retention policies, or bounded subsets.

80. Read concern, write concern, transactions, replica sets.
- Read concern defines consistency guarantees for reads.
- Write concern defines the acknowledgement level for writes.
- Transactions allow multiple operations to commit or abort atomically when supported.
- Replica sets keep copies of data and provide redundancy/failover.
Choose guarantees based on correctness and latency/availability needs; transactions do not replace good schema design.

81. skip/limit versus range pagination.
skip/limit is simple but deep skips can be expensive. Range pagination uses an indexed stable sort key and a cursor condition, e.g. orderDate plus _id, to fetch the next page. It generally scales better for deep pages. Use a deterministic sort.

82. Prevent duplicate orders when an API request is retried.
Use a client/request idempotency key and enforce uniqueness with a unique index. On retry, return the result associated with the same key rather than creating a second order. Validate payload consistency and consider transaction boundaries for order, inventory, and payment workflows.

83. Aggregation versus MapReduce.
MongoDB's aggregation framework is generally preferred because it is expressive and optimized with native stages/operators. MapReduce is deprecated in modern MongoDB releases. Use aggregation for typical database transformations and reports; use application-side processing when it better fits the task.

84. Explain sharding and shard keys.
Sharding distributes a collection across multiple shards. The shard key controls distribution and query targeting. A low-cardinality or monotonically increasing key may create hotspots; a key poorly aligned with query patterns may force scatter-gather queries. Choose based on cardinality, write distribution, and common queries.

============================================================
G. SCHEMA DESIGN AND INTERVIEW SCENARIOS
============================================================

85. Design customers, products, orders, and reviews.
- customers: profile fields in customers.
- products: current catalogue fields in products.
- orders: customerId reference; embed bounded items with productId, purchase-time name/price snapshot, and quantity.
- reviews: productId and customerId references; store rating, text, and date.
Embed bounded data read together. Reference independently updated or unbounded data. Add indexes based on access patterns, e.g. orders {customerId:1, orderDate:-1} and reviews {productId:1}.

86. Live dashboard or summary collection?
For modest workloads, live aggregation may be sufficient. If reports are frequent/expensive and some staleness is acceptable, maintain a materialized summary collection using scheduled jobs, change streams, or application logic. Define freshness, rebuild/backfill, and consistency rules; measure before optimizing.

87. Prevent growth for thousands of order items or millions of events.
Do not put unbounded data in one parent array. Store items/events in a separate collection, possibly bucketed by parent and time. Keep parent documents small; query children using indexed references. Plan pagination, retention, and aggregation.

88. Preserve historical prices after product price changes.
Store purchase-time unitPrice and any required descriptive snapshot on each order line. Do not recalculate past orders using current product price. Product documents represent current catalogue values; orders preserve purchase-time facts.

89. Aggregation has duplicate totals after $lookup.
A one-to-many $lookup followed by $unwind multiplies parent rows. Summing an order-level total after that may count it once per child. Aggregate child records before joining, group at the correct level, or sum line-item values rather than repeating parent totals.

90. Fast locally but slow in production: what to inspect?
Inspect production data volume/distribution, indexes, explain("executionStats"), working-set/cache, CPU/disk, network latency, concurrency, write load, selectivity, aggregation spills, and deployment topology. Reproduce with representative data and compare query plans rather than assuming application code is the only cause.

91. Missing, null, and empty array differences.
- Missing: field does not exist.
- Null: field exists with null value.
- Empty array: field exists as [].
{field:null} matches explicit null and missing. $unwind drops missing/null/empty arrays by default unless preserving them. $size requires an array and may error on other types; guard with $isArray/$cond.

92. Clarify “top-selling product.”
Ask whether it means number of orders containing it, total units sold, revenue, or profit. Clarify time window, order statuses, returns/cancellations, discounts, and tax/shipping rules before writing the query.

93. High write rate and many indexes: trade-off.
Indexes improve supported reads and sorts but use memory/disk and add work to inserts, updates, and deletes. Keep indexes that support real query patterns, inspect index usage, remove redundant indexes carefully, and measure read/write trade-offs.

94. Include customers with zero orders in a report.
Start from customers, then $lookup orders using customers._id to orders.customerId. Keep the empty array or aggregate inside the lookup pipeline. Use $ifNull/default values so zero-order customers show count 0 and spend 0. Starting from orders omits customers with no orders.

95. Detect and safely migrate mixed string/numeric prices.
Detect BSON types:
db.products.aggregate([
  { $group: { _id: { $type: "$price" }, count: { $sum: 1 } } }
])
Test conversion:
db.products.aggregate([
  { $set: {
      convertedPrice: {
        $convert: {
          input: "$price", to: "decimal",
          onError: null, onNull: null
        }
      }
  }}
])
Review failed conversions before updating production. Back up, validate ranges/currency/precision, migrate in batches, add schema validation, and monitor the rollout. Decimal128 or integer minor units are usually safer for money than binary floating point.

============================================================
QUICK REFERENCE
============================================================

Filter:          $match
Select/compute:  $project, $set, $addFields
Group:           $group, $sum, $avg, $min, $max
Sort/paginate:   $sort, $skip, $limit
Array expansion: $unwind
Join:            $lookup
Multiple reports:$facet
Price bands:     $bucket, $bucketAuto
Conditional:     $cond, $switch
Array operators: $map, $filter, $reduce, $size, $arrayElemAt
Null/type:       $ifNull, $type, $isArray, $convert
Window ranking:  $setWindowFields, $rank
Persist results: $merge, $out

HOW TO EXPLAIN AN AGGREGATION IN AN INTERVIEW
1. Input grain: what one document represents before the pipeline.
2. Stage logic: what each stage filters, expands, groups, or calculates.
3. Output grain: what one result document represents.
4. Business assumptions: statuses, date range, discounts, refunds, and tax.
5. Performance: useful indexes, cardinality, and whether $unwind/$lookup multiplies rows.

END OF ANSWER KEY
"""

out.write_text(content, encoding="utf-8")
print(f"Created downloadable text file: {out}")
print(f"Size: {out.stat().st_size:,} bytes")
