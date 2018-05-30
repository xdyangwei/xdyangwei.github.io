## MongoDB基本教程

MongoDB是典型的NoSQL数据库，即非关系型数据库。MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。

下面将介绍MongoDB的最常用操作

### 1.开启MongoDB服务

在MongoDB目录下的bin目录下使用`sudo ./mongod`即可开启MongoDB服务

### 2.打开MongoDB命令行交互界面

在将bin目录添加进系统PATH路径后[^1]，任意目录输入`mongo`即可打开命令行交互界面

### 3.创建数据库

语法格式如下

```mo
use dbname
```

dnname就是你想创建的数据库的名字

**PS:如果同名数据库已存在，则系统会切换到这一数据库，而不是再创建一个同名数据库**

### 4.查看所有数据库

	show dbs
**PS:新建的没有记录的数据库不会显示**

### 5.删除数据库

想要删除数据库首先需要切换到这一数据库，然后对其进行删除操作

	use dbname
	db.dropDatabase()
### 6.创建集合

MongoDB中的集合collection相对应的就是RDBMS关系型数据库中的table,具体操作需要切换到指定数据库再创建集合，语法如下

	use dbname
	db.createCollection("collection_name",options)

其中options包含参数有：

**options**:**capped**:（可选）布尔类型，如果为 true，则创建固定集合。固定集合是指		                                                                                                                                                                         有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。
                                **当该值为 true 时，必须指定 size 参数。**
       **autoIndexId**:（可选）布尔类型，如为 true，自动在 _id 字段创建索引。默认为 false。
                      **size**: (可选) 数值类型，为固定集合指定一个最大值（以字节计）。
                               **如果 capped 为 true，也需要指定该字段。**
                      **max**: (可选) 数值类型，指定固定集合中包含文档的最大数量。

另一种创建集合的方法是直接往想创建的集合中插入文档即可。

	use dbname
	db.collection_name.insert({"name":"yangwei"})

这样我们既创建了集合，也往这个新建的集合中插入了我们的文档

### 7.查看集合

	use dbname
	show collections/tables

### 8.删除集合

	use dbname
	db.collection_name.drop()

### 9.插入文档

	use dbname
	db.collection_name.insert(document)

由于MongoDB是非关系型数据库，MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。 形如：

	> document=({title: 'MongoDB 教程', 
	description: 'MongoDB 是一个 Nosql 数据库',
	by: 'yangwei',
	url: 'xdyangwei.github.io',
	tags: ['mongodb', 'database', 'NoSQL'],
	likes: 100
	});
还有几种语法：

	db.collection_name.insertOne():向指定集合中插入一条文档数据
	db.collection_name.insertMany():向指定集合中插入多条文档数据

### 10.更新文档

MongoDB 使用 **update()** 和 **save()** 方法来更新集合中的文档。 

#### update() 方法

update()方法用于更新已存在的文档。语法格式如下：

	db.collection_name.update(
	   <query>,
	   <update>,
	   {
	     upsert: <boolean>,
	     multi: <boolean>,
	     writeConcern: <document>
	   }
	)
**参数说明**：

- **query** : update的查询条件，类似sql update查询内where后面的。

- **update** : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的

- **upsert** : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。

- **multi** : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。

- **writeConcern** :可选，抛出异常的级别。

**实例：**
	db.collection_name.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}})

#### save() 方法

save() 方法通过传入的文档来替换已有文档。语法格式如下： 

	db.collection_name.save(
	<document>,
	{
	 writeConcern: <document>
	}
	)

**实例：**

替换 _id 为 56064f89ade2f21f36b03136 的文档数据 

	>db.collection_name.save({
	"_id" : ObjectId("56064f89ade2f21f36b03136"),
	"title" : "MongoDB",
	"description" : "MongoDB 是一个 Nosql 数据库",
	"by" : "yangwei",
	"url" : "xdyangwei.github.io",
	"tags" : [
	        "mongodb",
	        "NoSQL"
	],
	"likes" : 110
	})
### 11.删除文档

MongoDB remove()函数是用来移除集合中的数据。 基本语法格式如下：

	db.collection_name.remove(
	   <query>,
	   {
	     justOne: <boolean>,
	     writeConcern: <document>
	   }
	)
**参数说明：**

- **query** :（可选）删除的文档的条件。

- **justOne** : （可选）如果设为 true 或 1，则只删除一个文档。

- **writeConcern** :（可选）抛出异常的级别。

**Tips:在执行remove()函数前先执行find()命令来判断执行的条件是否正确** 

### 12.查询文档

MongoDB 查询文档使用 find() 方法。 语法格式如下：

	db.collection_name.find(query, projection)

- **query** ：可选，使用查询操作符指定查询条件
- **projection** ：可选，使用投影操作符指定返回的键。查询时返回文档中所有键值， 只需省略该参数即可（默认省略）。

若不指定 projection，则默认返回所有键，指定 projection 格式如下，有两种模式 

	db.collection.find(query, {title: 1, by: 1}) // inclusion模式 指定返回的键，不返回其他键
	db.collection.find(query, {title: 0, by: 0}) // exclusion模式 指定不返回的键,返回其他键

_id 键默认返回，需要主动指定 _id:0 才会隐藏
两种模式不可混用（因为这样的话无法推断其他键是否应返回）

这时你会发现文档显示是以非结构化的方式显示，不易读，如果想要以结构化易读的方式显示，可以使用pretty()方法，如下：

	db.collection_name.find().pretty()

#### MongoDB中的查询条件

| 操作       | 格式                   | 范例                                      | RDBMS中的类似语句    |
| ---------- | ---------------------- | ----------------------------------------- | -------------------- |
| 等于       | `{<key>:<value>`}      | db.col.find({"by":"yangwei"}).pretty()    | where by = 'yangwie' |
| 小于       | {<key>:{$lt:<value>}}  | db.col.find({"likes":{$lt:50}}).pretty()  | where likes < 50     |
| 小于或等于 | {<key>:{$lte:<value>}} | db.col.find({"likes":{$lte:50}}).pretty() | where likes <= 50    |
| 大于       | {<key>:{$gt:<value>}}  | db.col.find({"likes":{$gt:50}}).pretty()  | where likes > 50     |
| 大于或等于 | {<key>:{$gte:<value>}} | db.col.find({"likes":{$gte:50}}).pretty() | where likes >= 50    |
| 不等于     | {<key>:{$ne:<value>}}  | db.col.find({"likes":{$ne:50}}).pretty()  | where likes != 50    |

#### AND条件

MongoDB 的 find() 方法可以传入多个键(key)，每个键(key)以逗号隔开，即常规 SQL 的 AND 条件。 

语法格式如下:

	db.collection_name.find({key1:value1, key2:value2}).pretty()

#### OR条件

MongoDB OR 条件语句使用了关键字 **$or**,语法格式如下： 

	db.col.find(
	{
	  $or: [
	     {key1: value1}, {key2:value2}
	  ]
	}
	).pretty()







[^1]: 打开系统主目录下的 .bashrc文件，在最下方加入`export PATH=<mongodb-install-directory>/bin:$PATH`即可