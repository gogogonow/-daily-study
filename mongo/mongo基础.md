<a name="B5mxg"></a>
# 一、文档（Documents）
文档为mongo的核心，为mongo基本存储形式，一个文档就是一个对象，类似于关系型数据库一行数据。如下：<br />{"name" : "klaus"}<br />就是一个文档，由是一个对象，有key（name）作为对象属性的键，value（klaus）为属性的值。<br />**注意：**任何UTF-8的字符串都可以作为key，但有以下约束：

1. null（\0）不能作为key
1. **.**和**$**有特殊属性，通常不作为key，因此只能在特定场景使用
1. 同一个文档中不能存在相同的key，同时mongo对大小写和类型敏感
<a name="NldML"></a>
## 1.1 插入文档
<a name="q6f6k"></a>
### 1.1.1 insertOne
```shell
db.movies.insertOne({"title" : "Stand by Me"})
```
<a name="dD5GR"></a>
### 1.1.2 inertMany
```shell
db.movies.drop()
db.movies.insertMany([
{"title" : "Ghostbusters"},
{"title" : "E.T."},
{"title" : "Blade Runner"}]);
db.movies.find()
```
注意：<br />insertMany支持第二个参数指定是否按顺序插入，不指定默认按顺序插入，如下：
```shell
db.movies.insertMany([
{"_id" : 0, "title" : "Top Gun"},
{"_id" : 1, "title" : "Back to the Future"},
{"_id" : 1, "title" : "Gremlins"},
{"_id" : 2, "title" : "Aliens"}])
```
上面没指定顺序，默认按顺序插入，因此在插入第三个文档时，由于_id字段重复导致插入失败，由于按顺序，后面的数据就会中断插入操作；<br />如果指定了不按顺序插入，即使中途插入失败也不会影响后续的数据插入：
```shell
db.movies.insertMany([
{"_id" : 3, "title" : "Sixteen Candles"},
{"_id" : 4, "title" : "The Terminator"},
{"_id" : 4, "title" : "The Princess Bride"},
{"_id" : 5, "title" : "Scarface"}],
{"ordered" : false})
```
<a name="LXMJU"></a>
### 1.1.3 插入校验
插入文档为了保证文档数据性能和设计合理性，会校验单文档最大不超过16M，可使用Object.bsonsize(doc)查看文档大小。
<a name="TIW5f"></a>
## 1.2 删除文档
<a name="tDX0q"></a>
### 1.2.1 deleteOne
删除一个文档，即使根据删除条件会匹配到多个文档，也只会删除找到的第一个文档
```shell
db.movies.deleteOne("_id" : 4})
```
<a name="BkcEd"></a>
### 1.2.2 deleteMany
删除匹配的所有文档
```shell
db.movies.deleteMany({"year" : 1984})
```
删除所有
```shell
db.movies.deleteMany({})
```
<a name="hQjXx"></a>
### 1.2.3 drop
删除集合
```shell
db.movies.drop()
```
<a name="u8jgs"></a>
## 1.3 更新
<a name="YTmvY"></a>
### 1.3.1 replaceOne\replaceMany
替换匹配文档为新的文档，通常用于重组文档结构<br />注意：查找一个唯一文档，修改内容，根据条件替换，条件查找到非唯一文档，但是都替换成查找到的唯一文档的_id，会导致报_id重复异常
<a name="bQjwA"></a>
### 1.3.2 updateOne\updateMany
更新匹配到的第一个文档，第一个参数为更新条件，第二个为更新数据<br />**$set**可设置字段，该字段没有则会新创建<br />**$unset**可删除字段，该字段会变为不存在<br />**$inc**可做加算数操作，和$set类似，即使字段不存在也会创建<br />**$push**可对数组添加元素，添加到数组尾部，数组不存在则会创建<br />**$each**支持一次push多个数据到数组中
```shell
db.students.updateOne(
   { _id: 2 },
   {
     $push: {
       scores: {
         $each: [ 100, 20 ],
         $slice: 3
       }
     }
   }
)
```
**$slice**元素会在进行$push操作的时候限制数组元素的个数，$slice修饰符必须和$each修饰符一起使用，你也可以传递一个空的数组给$each修饰符，因此只有$slice修饰符起作用。负数表示从后往前，反之为正数表示从前往后。<br />原始文档：
```shell
{ "_id" : 2, "scores" : [ 89, 90 ] }
```
执行更新：
```shell
db.students.updateOne(
   { _id: 2 },
   {
     $push: {
       scores: {
         $each: [ 100, 20 ],
         $slice: 3
       }
     }
   }
)
```
操作结果：
```shell
{ "_id" : 2, "scores" : [  89,  90,  100 ] }
```
**$addToSet**将不存在的数据添加到数组中<br />**$pop**从数组末尾移除元素，可指定正负数标识从首部{"$pop" : {"key" : -1}}还是尾部{"$pop" : {"key" : 1}}移除<br />**$pull**从数组中移除匹配的元素<br />**指定数组位置修改**，通过数组下标索引指定修改数据，如果不知道在哪个索引位置，可使用$表示匹配的第一个，如下把第一个comments.author为John的改为Jim
```shell
db.blog.updateOne({"comments.author" : "John"}, {"$set" : {"comments.$.author" : "Jim"}})
```
<a name="fyRV4"></a>
### 1.3.3 Upserts
特殊的更新操作，无匹配的文档就会新创建文档
```shell
{"upsert" : true}
```
是一个更新选项，不存在就插入
<a name="CqJBk"></a>
### 1.3.4 返回更新的文档
早期版本支持findAndModify<br />后续版本为了使用更简单支持findOneAndDelete、findOneAndReplace、findOneAndUpdate
<a name="Ua1LB"></a>
# 二、集合（Collections）
集合就是一组文档，类似于关系型数据库的表，但与RDB不同的是集合支持动态结构（不同文档间keys、keys数量、values的类型都可以不同）<br />此处可能会有疑问，既然集合的内部结构可以不一样，那为啥还要区分不同的集合，直接放一个多方便！<br />我认为有以下几个原因：

1. 通常我们使用编程语言和mongo交互，而编程语言中的数据类结构都是基本固定的，因此需要区分集合
1. 将所有文档都放到同一个集合不便于管理，比如不便于建立索引，建立索引低效等
1. 多集合划分能提高程序CRUD性能
<a name="RL9Wu"></a>
## 2.1 集合命名约束（Naming）
支持任何UTF-8的字符串，有以下约束：

1. 不能为空字符串
1. 不能包含**\0**，其标识集合名称结尾
1. 不能以**system.**为前缀，其为保留前缀
1. 不能包含**$**，因为在访问集合时通过**$**取某个集合
<a name="fwhah"></a>
## 2.2 子集合
之和通过**.**获取，例如db.users.scores即为users集合的子集合scores
<a name="AdcbZ"></a>
# 三、数据库（Databases）
数据库包含一组集合，类似RDB的数据库，数据库命名也支持任何UTF-8的字符串，有以下约束：

1. 不能为空字符串
1. 不能包含 **/, \, ., ", *, <, >, :, |, ?, $, 单个空格, \0**
1. 大小写敏感
1. 长度<= 64字节
1. 以下为保留名称，**admin, local, config**
<a name="xQXcA"></a>
# 四、Mongo Shell
mongo shell使用mongo命令进入，支持js语言，内部包含js语言解释器<br />基本命令：<br />show dbs
> 显示数据库

show collections
> 显示集合

show users
> 显示用户

mongo script.js
> 运行脚本

<a name="ikMV0"></a>
# 五、数据类型
<a name="RLhbi"></a>
## 5.1 基础类型
Null: 表示null值或不存在的字段<br />Boolean: 表示值true或false<br />Number: mongo shell默认使用64位浮点型，可使用NumberInt("XXX")表示4字节整型，NumberLong("XXX")表示8字节长整型<br />String: 字符串<br />Date: 日期，64位的整数存储1970-1-1后的毫秒数，不会存储时区信息<br />Regular Expression: 正则表达式<br />Array: 数组[]<br />嵌套文档: 文档对象内部嵌套对象<br />Object ID: 12字节的id，标识文档<br />Code: 代码段<br />注意：ObjectId和_id关系？
> 每个文档都有_id，每个集合内部文档的_id都是唯一的，ObjectId为_id的默认类型，为mongo分片集群生成唯一id，为字符串形式，按时间递增，包含12字节，24个16进制字符，前四字节存储时间戳（1970-1-1的秒数），中间五字节为随机值，避免不同机器生成相同id，最后三字节为计数器，为了保证同一个机器同一秒内生成的id不一样，3个字节也就是2563 支持16777216个id生成。

<a name="cyTVw"></a>
# 六、查询
mongo文档查询使用db.collection.find()，find无参数表示查询集合中所有文档，第一个参数为查询条件，第二个参数可指定查询字段。
<a name="bj0i9"></a>
## 6.1 特定字段和值查询
字段匹配：
```shell
db.users.find({"username" : "joe", "age" : 27})
```
筛选返回字段：
```shell
db.users.find({}, {"username" : 1, "_id" : 0})
```
<a name="H1gqL"></a>
## 6.2 查询条件
$lt (<)	$lte (<=)	$gt (>)		$gte (>=)<br />$or		$in		$nin	$ne		$eq		$not<br />例：
```shell
db.users.find({"username" : {"$ne" : "joe"}}) 
db.users.find({"user_id" : {"$in" : [12345, "joe"]}}) 
db.users.find({"id_num" : {"$mod" : [5, 1]}}) 
db.users.find({"id_num" : {"$not" : {"$mod" : [5, 1]}}})
```
<a name="U5PYJ"></a>
## 6.3 指定类型查询
null表示值为null或者字段不存在两层含义，如果要查询某个字段值为null可结合$exists : true
<a name="iihME"></a>
## 6.4 正则表达式查询
```shell
db.users.find({"name" : {"$regex" : /joe/i } })
```
查询name时joe且对大小写不敏感的文档。<br />mongo正则可使用JavaScript shell测试后使用。
<a name="DwLxd"></a>
## 6.5 查询数组
数组中单一元素条件查询（只要有该元素即匹配）：<br />插入数据：
```shell
db.food.insertOne({"fruit" : ["apple", "banana", "peach"]}) 
```
查询：
```shell
db.food.find({"fruit" : "banana"}) 
```

$all 查询数组中包含apple和banana元素的所有数据（只要数字化总有对应元素即可）：<br />插入数据：
```shell
a、db.food.insertOne({"_id" : 1, "fruit" : ["apple", "banana", "peach"]}) 
b、db.food.insertOne({"_id" : 2, "fruit" : ["apple", "kumquat", "orange"]}) 
c、db.food.insertOne({"_id" : 3, "fruit" : ["cherry", "banana", "apple"]}) 
```
查询：
```shell
db.food.find({fruit : {$all : ["apple", "banana"]}})
```
返回：a、c

数组精准匹配查询（需要数组条件中元素顺序和值都完全匹配）：<br />查询：
```shell
db.food.find({"fruit" : ["apple", "banana", "peach"]})
```
返回：a

$size 查询数组大小为3的：<br />查询：
```shell
db.food.find({"fruit" : {"$size" : 3}})
```

$slice查询数组首部或尾部元素个数，整数表示首部开始，负数表示尾部开始<br />查询前10个comments：
```shell
db.blog.posts.findOne({}, {"comments" : {"$slice" : 10}}) 
```
查询尾部后10个comments：
```shell
db.blog.posts.findOne({}, {"comments" : {"$slice" : -10}}) 
```
跳过前23个，一共返回10个：
```shell
db.blog.posts.findOne({}, {"comments" : {"$slice" : [23, 10]}})
```

<a name="jLSLc"></a>
## 6.6 嵌套查询
以下两种效果一样：
```shell
db.people.find({"name" : {"first" : "Joe", "last" : "Schmoe"}}) 
db.people.find({"name.first" : "Joe", "name.last" : "Schmoe"})
```

<a name="L1Giq"></a>
## 6.7 $where js作为条件查询
插入数据：
```shell
a、db.foo.insertOne({"apple" : 1, "banana" : 6, "peach" : 3}) 
b、db.foo.insertOne({"apple" : 8, "spinach" : 4, "watermelon" : 4})
```
查询文档中有任何两个字段值相同的数据：
```shell
db.foo.find({"$where" : function () { 
	for (var current in this) { 
		for (var other in this) {
			if (current != other && this[current] == this[other]) 
				{ return true; } 
		}
	}
	return false;
}});
```
返回：b<br />b中spinach和watermelon的值都是4

<a name="kt1LV"></a>
## 6.8 游标
游标相当于C语言的指针，可以定位到某个文档。通过游标能实现限制结果数量，跳过部分结果或根据任意键按任意顺序的组合对结果进行各种排序等。<br />例：
```shell
var mycursor = db.bar.find({_id:{$lte:5}})
while(mycursor.hasNext()) {
    printjson(mycursor.next());
}
```
<a name="oAv36"></a>
## 6.9 Limits、Skips、Sorts
```shell
db.foo.find().skip(10).limit(1).sort({"x" : 1});
```
注意：<br />1、当sort,skip,limit一起使用时，无论其位置变化，总是先sort再skip，最后limit。如果想不按照默认顺序，可使用aggregate聚合，聚合使用流的方式，会依次执行条件。<br />2、不要使用skip跳过大数据，性能低<br />mongo混合类型排序规则：<br />Null < Numbers < Strings < Object/document < Array < Binary data < Object ID <Boolean < Date <Timestamp < Regular expression


