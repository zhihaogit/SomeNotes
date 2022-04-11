# MongoDB

## 基础

### 简介

- MongoDB是为快速开发互联网 web应用而设计的数据库系统
- MongoDB的设计目的是极简、灵活、作为 web应用栈的一部分
- MongoDB的数据模型是面向文档的，所谓文档是一种类似于 JSON的结构，简单理解 MongoDB这个数据库中存的是各种JSON。(BSON)

### 三个概念

- 数据库（database）是一个仓库，在仓库中可以存放集合。不需要手动创建，切换即创建
- 集合（Collection）集合类似于数组，在集合中可以存放文档。不需要手动创建，切换即创建
- 文档（Document）是数据库中的最小单位，我们存储和操作的内容都是文档

### 基本指令

```sql
show dbs  --  显示当前的所有数据库
show databases --  显示当前的所有数据库
use 数据库 -- 进入到指令的数据库中
db -- 显示当前的数据库名称

show collections -- 显示数据库中所有的集合  
```

### CRUD操作

#### 新增

```sql
-- 向集合中插入一个或多个文档
db.<collection>.insert(JSONObject|JSONArray)
db.<collection>.insertOne(JSONObject)
db.<collection>.insertMany(JSONArray)
-- 当向集合中插入文档时，如果没有指定 _id属性，数据库则会自动为文档添加 _id，该属性用来作为文档的唯一标识

-- 手动生成 ObjectId()
ObjectId()


-- 例子：
db.stus.insert({name: "haha", age: 18});
db.stus.insert([
  {name: "haha1", age: 18},
  {_id: "hello", name: "haha2", age: 19}
]);
db.stus.insertOne({name: "haha", age: 18});
db.stus.insertMany([
  {name: "haha3", age: 18},
  {name: "haha4", age: 19}
]);
```

#### 查询

```sql
-- 查询当前集合中所有的文档
db.<collection>.find();
-- find()可以接收一个对象作为条件参数
-- find({})也是查找所有文档
-- find({属性: 值}) 查询属性是指定值的文档
-- find()返回的是一个数组

-- 查询所有结果的数量
db.<collection>.find({}).count();
db.<collection>.find({}).length();

-- limit查询
db.<collection>.find({}).limit(n);

-- 分页查询
-- limit()每页显示的条数
-- skip()跳过前多少条，分页数据的起始点，n = (页码 - 1) * 每页显示的条数
db.<collection>.find({}).skip(n).limit(n);

-- 用来查询集合中符合条件的第一个文档
db.<collection>.findOne();
-- 返回的是一个文档对象


-- 例子：
db.stus.find({_id: "hello"});
db.stus.find({name: "haha1", age: 18});
db.stus.findOne({age: 18});

-- MongoDB支持直接通过内嵌文档的属性进行查询，如果要查询内嵌文档则可以通过 .的形式来匹配
-- 如果要通过内嵌文档进行查询，此时属性名必须使用双引号
db.users.find({"hobby.movies": "hero"});

-- 循环添加数据
-- 方式一：循环调用 insert
for (var i = 1; i <= 20000; i++) {
	db.numbers.insert({num: i});
}
-- 方式二：生成数据后，执行一次 insert
var arr = [];
for (var i = 1; i <= 20000; i++) {
	arr.push({num: i});
}
db.numbers.insert(arr);

-- 大于小于等于查询条件
db.numbers.find({num: {$gt: 35000}}); -- 大于 35000
db.numbers.find({num: {$lt: 30}});  -- 小于 30
db.numbers.find({num: {$eq: 1000}}); -- 等于 1000
db.numbers.find({num: {$gt: 10, $lt: 20}}); -- 大于 10，小于 20
db.numbers.find({num: {$lte: 10}});  -- 小于等于 10

-- 分页查询，前11 - 20之间的数据
db.numbers.find({}).skip(10).limit(10);

-- 查询工资小于 2000的员工
db.emp.find({sal: {$lt: 2000}});
-- 查询工资小于 1000或大于 2500的员工
db.emp.find({$or: [{sal: {$lt: 1000}}, {sal: {$gt: 2500}}]});
```

#### 修改

```sql
-- 修改匹配到的数据
db.<collection>.update(查询条件, 新对象 [, 标识符]);
-- 默认情况下会使用新对象直接覆盖第一个匹配数据中的旧对象
-- 如果需要修改指定的属性，而不是覆盖，需要使用“修改标识符”来完成修改
   -- $set 可以用来修改文档中的指定属性
   -- $unset 可以用来删除文档中的指定属性
   -- $push 向数组中添加一个新元素
   -- $addToSet 向数组中添加一个新元素，不重复添加
   -- $inc 在原始数据中自增指定大小
-- 标识符，{multi: true}同时修改所有符合条件的文档

-- 同时修改多个符合条件的文档
db.<collection>.updateMany()

-- 修改一个符合条件的文档
db.<collection>.updateOne()

-- 替换一个文档
db.<collection>.replaceOne()


-- 例子：
-- 直接替换 _id=hello的数据
db.stus.update(
  {_id: "hello"},
  {age: 188}
);
-- 只修改 age字段值
db.stus.update(
  {name: "haha1"},
  {
    $set: {
      age: 188,
      gender: "male"
    }
  }
);
-- 删除 gender字段
db.stus.update(
  {name: "haha1"},
  {
  	$unset: { gender: "male" }
  }
);
-- 同时修改多个数据
db.stus.update(
  {age: 18},
  {
  	$set: { gender: "male" }
  },
  { multi: true }
);
db.users.update(
	{username: "ts"},
  {
    $push: {
			"hobby.movies": "Interstellar"
    }
  }
)

-- 为所有薪资低于1000的员工增加工资400块
db.emp.updateMany({sal: {$lte: 1000}}, {$inc: {sal: 400}});
```

#### 删除

```sql
-- 删除数据，默认删除所有符合条件的数据
db.<collection>.remove()
-- 删除一个或多个，第二参数传递 true，只会删除一个
-- 如果传递一个空对象作为参数，则会清空集合

-- 删除一个
db.<collection>.deleteOne();

-- 删除多个
db.<collection>.deleteMany();

-- 删除集合，如果数据库只有一个集合，这个集合被删除时，数据库也会被删除
db.<collection>.drop();

-- 删除数据库
db.dropDatabase()


-- 例子：
-- 删除所有符合 _id=hello的数据
db.stus.remove({_id: "hello"});
-- 只删除一个 age=18的数据
db.stus.remove(
	{age: 18},
  true
);
```

### 文档之间的关系

#### 一对一（one to one）

例如：夫妻

在 mongoDB，可以通过**内嵌文档**的形式来体现出一对一的关系

```sql
db.wifeAndHusband.insert([
  {
    name: "wife1",
    husband: {
      name: "husband1"
    }
  },
  {
    name: "wife2",
    husband: {
      name: "husband2"
    }
  }
])
```

#### 一对多（one to many）|多对一（many to one）

例如：父母 对 孩子，用户 对 订单，文章 对 评论

- 可以通过**内嵌文档**的形式来实现

- 可以使用类似于**外键**的形式

  ```sql
  db.users.insert([
    {name: "swk", _id: "swk"},
    {name: "zbj", _id: "zbj"}
  ]);
  
  db.orders.insert([
    {
    	list: ["waterlemon", "orange", "blueberry"],
    	user_id: "swk"
    },
    {
    	list: ["apple", "mango", "strawberry"],
    	user_id: "zbj"
    }
  ]);
  
  var userId = db.users.findOne({ name: "swj" })._id;
  db.orders.find({user_id: userId});
  ```

#### 多对多（many to many）

例如：分类 对 商品，老师 对 学生

```sql
db.teachers.insert([
  {name: "hqg", _id: "hqg"},
  {name: "hys", _id: "hys"},
  {name: "ptlz", _id: "ptlz"}
]);

db.students.insert([
  {name: "gj", teacher_ids: [ "hqg", "hys" ]},
  {name: "swk", teacher_ids: [ "hqg", "hys", "ptlz" ]}
]);

db.teachers.find();
db.students.find();
```

### Sort

- 查询文档时，默认情况是按照 `_id`的值进行升序排列
- `sort()`可以用来指定文档的排序规则，需要一个对象形式的参数，`1`表示升序，`-1`表示降序
- `limit()`,`skip()`,`sort()`可以按任意的顺序进行调用

```sql
db.emp.find({}).sort({sal: 1, empno: -1});
```

### 投影

在查询时，可以在第二个参数的位置设置查询结果的投影，在查询结果中显示或隐藏相应列

```sql
-- 1 显示该字段，0 隐藏该字段
db.emp.find({}, {ename: 1, _id: 0, sal: 1});
```

### mongoose

> nodejs操作 mongo的 ODM（对象文档模型）库

#### 优势

- 可以为文档创建一个模式结构（Schema）
  - Schema对象定义约束了数据库中的文档结构
- 数据可以通过类型转换转换为对象模型（Model）
  - Model对象作为集合中的所有文档的表示，相当于MongoDB数据库中的集合 Collection
- 可以对模型中的对象/文档（Document）进行验证
  - Document表示集合中的具体文档，相当于集合中的一个具体文档
- 可以使用中间件来应用业务逻辑挂钩
- 比 node原生的 mongodb驱动更容易

### spring-boot-starter-data-mongodb

> spring项目使用的 mongodb操作包
