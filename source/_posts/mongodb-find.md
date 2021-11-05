---
title: mongodb查询
date: 2021-10-13 17:41:33
tags: [技术]
categories: mongodb
---

mongodb 基本查询

<!-- more -->



#### 新增

```
db.user.insert({"userName":"samz","age":12,"sex":1,"phone":"12345678909","roleIds":[1,4,,6,8,9,11]})
```

新增一个用户表(collection)，有字段userName、age、sex、phone、roleIds

#### 查询

mongodb查询格式为db.user.find()，括号内是查询语句用大括号包围。

1 、查询表全部信息

```
db.user.find(); 
```

2、 指定查询条件，查询userName是samz用户

```
db.user.find({"userName":"samz"}); 
```

3、 指定多个查询条件，查询userName是samz,性别是男的用户。

```
db.user.find({"userName":"samz","sex":1}); 
```



##### 分页排序

排序：

按照年龄降序排序。

db.user.find().sort({age:-1}); 	

分页查询

skip用于指定跳过记录数，limit则用于限定返回结果数量,

db.user.find().sort({age:-1}).skip(0).limit(8); 	

> skip是从0开始计算



##### 投射

投射（projection）可以让数据库只返回一部分被关注的字段，而不是整个文档

查询只返回用户名称

```
db.user.find({"userName":"samz"},{userName:1})
```

不返回_id字段

```
db.user.find({"userName":"samz"},{_id:0})
```

> 0 是不返回字段，1是只返回字段



##### 特殊条件查询

格式 { <field1>: { <operator1>: <value1> }, ... }



1、查询userName是samz用户

```
db.user.find({"userName":{"$eq":"samz"}}); 
```

2、查询age 大于18

```
db.user.find({"age":{"$gt":18}}); 
```

3、 查询age不是12

```
db.user.find({"age":{"$not":{"$eq":12}}});
```

> $not 值是个doc

4、 or 查询 

```
db.user.find({"userName":"samz",$or:[{"sex":1}]});
```

##### 操作符列表

- 比较操作符

| 操作符 | 说明             |
| ------ | ---------------- |
| $eq    | 等值比较         |
| $gt    | 大于指定值       |
| $gte   | 大于或等于指定值 |
| $in    | 数组中包含       |
| $lt    | 小于指定值       |
| $lte   | 小于或等于指定值 |
| $ne    | 不等于指定值     |
| $nin   | 不在数组中包含   |



- 逻辑操作符

mongodb原生查询介绍，

待写

<!-- more -->



| 操作符 | 说明 |
| ------ | ---- |
| $and   | 与   |
| $or    | 或   |
| $not   | 非   |
| $nor   | 即非 |



- 数组操作符

| 操作符     | 说明               |
| ---------- | ------------------ |
| $all       | 全包含             |
| $elemMatch | 仅包含一个元素匹配 |
| $size      | 大小匹配           |



##### 元素是否存在

{ field: { $exists: <boolean> } }

查询userName字段是否存在

db.user.find({"userName":{ $exists:true}});



#### 更新操作

操作语句

- db.collection.updateOne(<filter>, <update>, <options>)

- db.collection.updateMany(<filter>, <update>, <options>)

- db.collection.replaceOne(<filter>, <update>, <options>)

  

- 更新年龄

```
db.user.updateMany({"userName":"samz"},{$set:{"age":17}});
```



#### 删除操作

操作语句

- db.collection.deleteMany()

- db.collection.deleteOne()



1、 删除全部数据

```
db.user.deleteMany({})
```

1、 删除用户名为samz数据

```
db.user.deleteMany({"userName":"samz"})
```















