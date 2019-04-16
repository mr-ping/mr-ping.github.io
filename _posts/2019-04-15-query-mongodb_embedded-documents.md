---
title:  "MongoDB的嵌套查询 - 单个子文档"
date:   2019-04-15 20:43:43 +0800
categories: MongoDB
tags: 数据库 nosql 嵌套 查询 文档
---
MongoDB中的记录是基于文档的，文档中嵌套文档是常情。

下面，我们创建一个名为`inventory`的 `collection`，并插入几条嵌套有单个文档的记录，然后举例说明如何将其子文档做为过滤条件，对`inventory`中的记录进行查询。

```javascript
>>> db.inventory.insertMany( [
   { item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "A" },
   { item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
   { item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
   { item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" }
]);
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("5cb526ab8f34db1fbff6c1ed"),
		ObjectId("5cb526ab8f34db1fbff6c1ee"),
		ObjectId("5cb526ab8f34db1fbff6c1ef"),
		ObjectId("5cb526ab8f34db1fbff6c1f0"),
		ObjectId("5cb526ab8f34db1fbff6c1f1")
	]
}
```

集合`inventory`中的每条记录（如：`{ item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" }`）实际上都是一个单独的文档，其中`{ h: 14, w: 21, uom: "cm" }`是以*size*命名的一个子文档。

这种多层的嵌套结果相对关系型数据库来说结构更加清晰，易于阅读，但是也会操成查询方法相对关系型数据库来说不尽相同。

## 下面是两种查询方式：

### 第一种是将整个子文档当作查询条件：

此种查询要求查询条件中的子文档完全匹配实际记录，包括元素的顺序：

```javascript
>>> db.inventory.find( { size: { h: 14, w: 21, uom: "cm" } } )
{"_id":{"$oid":"5cb526ab8f34db1fbff6c1ed"},"item":"journal","qty":25,"size":{"h":14,"w":21,"uom":"cm"},"status":"A"}
```

上边查询条件与`inventory`中第一条记录的子文档完全匹配（包括顺序），因此可以返回第一条记录。

如果查询条件只匹配子文档中部分字段或者与子文档中顺序不符的话会怎样呢？

```javascript
>>> db.inventory.find(  { size: { w: 21 } }  )
[]
>>> db.inventory.find(  { size: { w: 21, h: 14, uom: "cm" } }  )
[]
```

很遗憾，无法查询到结果。

### 第二种是依据子文档中某个字段的值来进行查询

这种查询方式需要用 *点号* 将子文档的名字和想要查询的子文档字段连接起来，以字符串的形式作为查询条件的Key，如`"size.uom"`。

#### 精准匹配子文档中某个字段的值

这种查询方式，只需要在查询条件中指定子文档中某个字段的值即可。如果其值与子文档中的相应字段的值一致，即可返回结果：

```javascript
>>> db.inventory.find( { "size.uom": "in" } )
{"_id":{"$oid":"5cb526ab8f34db1fbff6c1ee"},"item":"notebook","qty":50,"size":{"h":8.5,"w":11,"uom":"in"},"status":"A"}
{"_id":{"$oid":"5cb526ab8f34db1fbff6c1ef"},"item":"paper","qty":100,"size":{"h":8.5,"w":11,"uom":"in"},"status":"D"}
```

`collection`中第3、4条记录，size子文档的`uom`字段值为`in`，查询返回第3、4条结果。

#### 使用请求运算符进行查询

对字段的匹配查询还可以使用*[请求运算符](https://docs.mongodb.com/manual/reference/operator/query/#query-selectors)*:

```javascript
>>> db.inventory.find( { "size.h": { $lt: 15 } } )
{"_id":{"$oid":"5cb526ab8f34db1fbff6c1ed"},"item":"journal","qty":25,"size":{"h":14,"w":21,"uom":"cm"},"status":"A"}
{"_id":{"$oid":"5cb526ab8f34db1fbff6c1ee"},"item":"notebook","qty":50,"size":{"h":8.5,"w":11,"uom":"in"},"status":"A"}
{"_id":{"$oid":"5cb526ab8f34db1fbff6c1ef"},"item":"paper","qty":100,"size":{"h":8.5,"w":11,"uom":"in"},"status":"D"}
{"_id":{"$oid":"5cb526ab8f34db1fbff6c1f1"},"item":"postcard","qty":45,"size":{"h":10,"w":15.25,"uom":"cm"},"status":"A"}
```

这条请求查询`size`子文档的`h`字段都小于15的记录，第1、3、4、5条符合条件并返回。

#### *逻辑与*在查询条件中的使用

如果希望结果同时满足多个过滤条件，除了使用*逻辑与*运算符（`$and`）外，还可以将过滤条件罗列在请求当中：

```javascript
// 将过滤条件罗列在 query 中查询
>>> db.inventory.find( { "size.h": { $lt: 15 }, "size.uom": "in", status: "D" } )
{"_id":{"$oid":"5cb526ab8f34db1fbff6c1ef"},"item":"paper","qty":100,"size":{"h":8.5,"w":11,"uom":"in"},"status":"D"}

// 通过 $and 请求运算符查询
>>> db.test1.find( {$and: [{"size.h": { $lt: 15 }}, {"size.uom": "in"}, {status: "D"}]})
{"_id":{"$oid":"5cb526ab8f34db1fbff6c1ef"},"item":"paper","qty":100,"size":{"h":8.5,"w":11,"uom":"in"},"status":"D"}
```

