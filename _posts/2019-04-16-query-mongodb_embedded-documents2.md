---
title:  "MongoDB的嵌套查询 - 子文档列表"
date:   2019-04-16 13:43:43 +0800
categories: MongoDB
tags: 数据库 nosql 嵌套 查询 文档 MongoDB
---

[上篇文章](http://mr-ping.com/mongodb/2019/04/15/query-mongodb_embedded-documents/)介绍的是含有单个子文档的MongoDB嵌套查询。  
这里再介绍下如何在含有多个子文档所组成的子文档列表的记录中，进行嵌套查询。

## 首先还是先建立个集合样本：

```javascript
>>> db.inventory.insertMany( [
   { item: "journal", instock: [ { warehouse: "A", qty: 5 }, { warehouse: "C", qty: 15 } ] },
   { item: "notebook", instock: [ { warehouse: "C", qty: 5 } ] },
   { item: "paper", instock: [ { warehouse: "A", qty: 60 }, { warehouse: "B", qty: 15 } ] },
   { item: "planner", instock: [ { warehouse: "A", qty: 40 }, { warehouse: "B", qty: 5 } ] },
   { item: "postcard", instock: [ { warehouse: "B", qty: 15 }, { warehouse: "C", qty: 35 } ] }
]);
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("5cb574858f34db1fbff6c1f2"),
		ObjectId("5cb574858f34db1fbff6c1f3"),
		ObjectId("5cb574858f34db1fbff6c1f4"),
		ObjectId("5cb574858f34db1fbff6c1f5"),
		ObjectId("5cb574858f34db1fbff6c1f6")
	]
}
```

上边建创建的`inventory`集合，每一条记录都嵌套有一个子文档列表，如第一条记录所在文档`{ item: "journal", instock: [ { warehouse: "A", qty: 5 }, { warehouse: "C", qty: 15 } ] }`中嵌套着含有`{ warehouse: "A", qty: 5 }`和`{ warehouse: "C", qty: 15 }`两个子文档，名为`instock`的列表。

## 接下来看一下查询方式：

### 查询条件与列表中的子文档完全匹配：

*如果记录中的子文档列表内有任何一个子文档与查询条件完全匹配（子文档元素顺序也需要一致），那么此记录视为符合查询条件:*

```javascript
>>> db.inventory.find( { "instock": { warehouse: "A", qty: 5 } } )
{"_id":{"$oid":"5cb574858f34db1fbff6c1f2"},"item":"journal","instock":[{"warehouse":"A","qty":5},{"warehouse":"C","qty":15}]}
```

如果部分匹配或者顺序不一致，则无法查询到结果：

```javascript
>>> db.inventory.find( { "instock": { warehouse: "A" } } )
[]
>>> db.inventory.find( { "instock": { qty: 5, warehouse: "A" } } )
[]
```



### 根据子文档字段值进行匹配

*用如下方式进行查询，如果子文档列表中任何一个子文档的字段满足条件，都可以匹配到。*

比如，以下命令中，只要`instock`列表内任意一个子文档的`qty`字段小于等于**20**，都视为满足条件：

```javascript
>>> db.inventory.find( { 'instock.qty': { $lte: 20 } } )
{"_id":{"$oid":"5cb574858f34db1fbff6c1f2"},"item":"journal","instock":[{"warehouse":"A","qty":5},{"warehouse":"C","qty":15}]}
{"_id":{"$oid":"5cb574858f34db1fbff6c1f3"},"item":"notebook","instock":[{"warehouse":"C","qty":5}]}
{"_id":{"$oid":"5cb574858f34db1fbff6c1f4"},"item":"paper","instock":[{"warehouse":"A","qty":60},{"warehouse":"B","qty":15}]}
{"_id":{"$oid":"5cb574858f34db1fbff6c1f5"},"item":"planner","instock":[{"warehouse":"A","qty":40},{"warehouse":"B","qty":5}]}
{"_id":{"$oid":"5cb574858f34db1fbff6c1f6"},"item":"postcard","instock":[{"warehouse":"B","qty":15},{"warehouse":"C","qty":35}]}
```

假设只想用子文档列表中的第一个子文档当作目标进行匹配呢？我们还可以通过如下方式指定目标子文档的索引值（从**0**开始）：

```javascript
>>> db.inventory.find( { 'instock.0.qty': { $lte: 20 } } )
{"_id":{"$oid":"5cb574858f34db1fbff6c1f2"},"item":"journal","instock":[{"warehouse":"A","qty":5},{"warehouse":"C","qty":15}]}
{"_id":{"$oid":"5cb574858f34db1fbff6c1f3"},"item":"notebook","instock":[{"warehouse":"C","qty":5}]}
{"_id":{"$oid":"5cb574858f34db1fbff6c1f6"},"item":"postcard","instock":[{"warehouse":"B","qty":15},{"warehouse":"C","qty":35}]}
```

指定索引值为**0**之后，只会拿子文档列表`instock`内的第一个文档进行匹配。

### 使用多个查询条件对子文档列表进行匹配

*使用`$elemMatch`操作符进行查询的话，要求子文档列表里至少有一个子文档完全满足`$elemMatch`内的所有条件。*

例如，下边命令要求，`instock`列表中至少有一个子文档同时满足`qty`值为**5**，并且`warehouse`值为**A**：

```javascript
>>> db.inventory.find( { "instock": { $elemMatch: { qty: 5, warehouse: "A" } } } )
{"_id":{"$oid":"5cb574858f34db1fbff6c1f2"},"item":"journal","instock":[{"warehouse":"A","qty":5},{"warehouse":"C","qty":15}]}
```

如下命令要求，`instock`列表中至少有一个子文档同时满足`qty`值大于**10**并且小于等于**20**：

```javascript
>>> db.inventory.find( { "instock": { $elemMatch: { qty: { $gt: 10, $lte: 20 } } } } )
{"_id":{"$oid":"5cb574858f34db1fbff6c1f2"},"item":"journal","instock":[{"warehouse":"A","qty":5},{"warehouse":"C","qty":15}]}
{"_id":{"$oid":"5cb574858f34db1fbff6c1f4"},"item":"paper","instock":[{"warehouse":"A","qty":60},{"warehouse":"B","qty":15}]}
{"_id":{"$oid":"5cb574858f34db1fbff6c1f6"},"item":"postcard","instock":[{"warehouse":"B","qty":15},{"warehouse":"C","qty":35}]}
```



*倘若不使用`$elemMatch`操作符，而是把多个查询条件作为组合，罗列在请求中的话，只要列表中有任何子文档（不必是同一个）匹配这些条件，即可返回。*

例如，下边的命令中，只要`instock`列表内有任意一个子文档满足`qty`字段大于**10**，并且任意一个子文档`qty`值小于等于**20**，即可视为满足条件。

```javascript
>>> db.inventory.find( { "instock.qty": { $gt: 10,  $lte: 20 } } )
{"_id":{"$oid":"5cb574858f34db1fbff6c1f2"},"item":"journal","instock":[{"warehouse":"A","qty":5},{"warehouse":"C","qty":15}]}
{"_id":{"$oid":"5cb574858f34db1fbff6c1f4"},"item":"paper","instock":[{"warehouse":"A","qty":60},{"warehouse":"B","qty":15}]}
{"_id":{"$oid":"5cb574858f34db1fbff6c1f5"},"item":"planner","instock":[{"warehouse":"A","qty":40},{"warehouse":"B","qty":5}]}
{"_id":{"$oid":"5cb574858f34db1fbff6c1f6"},"item":"postcard","instock":[{"warehouse":"B","qty":15},{"warehouse":"C","qty":35}]}
```

再比如，下边命令中，只要`instock`列表内有任意一个子文档满足`qty`字段值等于**5**，并且任意一个子文档`warehouse`值等于**A**，即视为满足条件。

```javascript
>>> db.inventory.find( { "instock.qty": 5, "instock.warehouse": "A" } )
{"_id":{"$oid":"5cb574858f34db1fbff6c1f2"},"item":"journal","instock":[{"warehouse":"A","qty":5},{"warehouse":"C","qty":15}]}
{"_id":{"$oid":"5cb574858f34db1fbff6c1f5"},"item":"planner","instock":[{"warehouse":"A","qty":40},{"warehouse":"B","qty":5}]}
```

