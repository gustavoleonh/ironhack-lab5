#

```

```

# ironhack-lab5

#### Implementing and Optimizing Database Queries

##### Task 1: SQL Query Optimization

###### 1.1 Orders

**Original query:**

``` sql
SELECT Orders.OrderID, SUM(OrderDetails.Quantity * OrderDetails.UnitPrice) AS TotalPrice
FROM Orders
JOIN OrderDetails ON Orders.OrderID = OrderDetails.OrderID
WHERE OrderDetails.Quantity > 10
GROUP BY Orders.OrderID;
```

**Optimization proposal:**

Two indexes are considered to be created in order to improve query performance.

``` sql
CREATE INDEX idx_orderdetails_orderid_quantity ON OrderDetails(OrderID, Quantity);
CREATE INDEX idx_orders_orderid ON Orders(OrderID);

SELECT 
    Orders.OrderID, 
    SUM(OrderDetails.Quantity * OrderDetails.UnitPrice) AS TotalPrice
FROM 
    Orders
JOIN 
    OrderDetails ON Orders.OrderID = OrderDetails.OrderID
WHERE 
    OrderDetails.Quantity > 10
GROUP BY 
    Orders.OrderID;
```

###### 1.2 Customer

**Original query:**

```
SELECT CustomerName FROM Customers WHERE City = 'London' ORDER BY CustomerName;
```

**Optimization proposal:**

``` sql
CREATE INDEX idx_customers_city ON Customers(City);
CREATE INDEX idx_customers_city_customername ON Customers(City, CustomerName);

SELECT CustomerName 
FROM Customers 
WHERE City = 'London' 
ORDER BY CustomerName;
```

##### Task 2: NoSQL Query Implementation

###### 2.1 OrdersUser Posts Query

**Original query:**

``` json
db.posts
  .find({ status: "active" }, { title: 1, likes: 1 })
  .sort({ likes: -1 });
```

**Optimization proposal:**


1. Create a compound index on status and likes fields. This will help the query filter by status and sort by likes efficiently.
2. Limit results.

``` json
db.posts.createIndex({ status: 1, likes: -1 });
db.posts.find({ status: "active" }, { title: 1, likes: 1 }).sort({ likes: -1 }).limit(100);
```

###### 2.2 User Data Aggregation:

**Original query:**

``` json
db.users.aggregate([
  { $match: { status: "active" } },
  { $group: { _id: "$location", totalUsers: { $sum: 1 } } },
]);
```

**Optimization proposal:**

1. Create indexes.
2. When quering results always use projection in order to only retriebve neccesary fields,
3. Ensure the `$match` stage is as specific as possible and appears early in the pipeline to reduce the number of documents processed in subsequent stages.

``` json
db.users.createIndex({ status: 1, location: 1 });
db.users.aggregate([
  { $match: { status: "active" } },
  { $group: { _id: "$location", totalUsers: { $sum: 1 } } },
]);
```
