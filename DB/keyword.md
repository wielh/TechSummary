# mongodb 對應的 mysql 關鍵字

## 目錄
查詢語句相關: WHERE, GROUP BY, HAVING, ORDER BY, LIMIT, DISTINCT
聯接與子查詢: JOIN，INNER JOIN，LEFT JOIN，RIGHT JOIN，UNION
事務控制: START TRANSACTION，COMMIT，ROLLBACK，SAVEPOINT，RELEASE SAVEPOINT

## 查詢範例
+ select 特定欄位
```
db.users.aggregate([
  { $project: { name: 1, city: 1} }
]);
```

+ WHERE, ORDER BY, LIMIT, DISTINCT 綜合範例
```
db.users.aggregate([
  { $match: { age: { $gt: 25 } } },        
  { $sort: { age: -1 } },                  
  { $skip: 100 },
  { $limit: 2 },                            
  { $group: { _id: "$city" } },             
  { $project: { city: "$_id", _id: 0 } }    
]);
```

+ GROUP BY, HAVING 綜合範例
```
db.sales.aggregate([
  {
    $group: {
      _id: "$item",
      total_quantity: { $sum: "$quantity" }
    }
  }, {
    $match: {
      total_quantity: { $gt: 20 }
    }
  }
]);
```

## 連接範例
+ inner join
```
db.orders.aggregate([
  {
    $lookup: {
      from: "customers", 
      localField: "customer_id", 
      foreignField: "_id", 
      as: "customer_info"
    }
  },
  { $unwind: "$customer_info" }
]);
```

```
db.orders.aggregate([
  // 第一次 JOIN：連接 customers 集合，按 customer_id
  {
    $lookup: {
      from: "customers",         // 要聯結的集合
      localField: "customer_id", // 當前集合中的鍵
      foreignField: "_id",       // customers 集合中的鍵
      as: "customer_info"        // 聯結結果存放到此字段
    }
  },
  { $unwind: "$customer_info" }, // 展開 customer_info

  // 第二次 JOIN：連接 products 集合，按 product_id
  {
    $lookup: {
      from: "products",          // 要聯結的集合
      localField: "product_id",  // 當前集合中的鍵
      foreignField: "_id",       // products 集合中的鍵
      as: "product_info"         // 聯結結果存放到此字段
    }
  },
  { $unwind: "$product_info" }, // 展開 product_info

  // 選擇需要的欄位輸出
  {
    $project: {
      _id: 1,
      total: 1,
      customer_name: "$customer_info.name",
      product_name: "$product_info.name"
    }
  }
]);
```

+ left join 
```
db.orders.aggregate([
  {
    $lookup: {
      from: "customers",
      localField: "customer_id",
      foreignField: "_id",
      as: "customer_info"
    }
  }
]);
```

+ union
db.customers.aggregate([
  { $project: { name: 1 } },
  { $unionWith: { coll: "employees", pipeline: [{ $project: { name: 1 } }] } }
]);