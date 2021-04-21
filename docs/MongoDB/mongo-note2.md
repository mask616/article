# MongoDB学习记录(二) #

## 更新操作(续) ##


> 更新数据  :

	注意:MongoDB使用update默认更新第一条数据	
	
	注释：如果后边的参数为true则表示，如果需要更新的记录不存在，则插入一条新的
	语法：db.[collection].update({"key":"value"},{$set:{"key":"value"}},true|false)
	
	例子：把name为“dozx”的数据的name字段值改为“ketty”，如果不存在则插入一条name为“ketty”的数据
		db.csbn.update({"name":"dozx"},{$set:{"name":"ketty"}},true)
	
	注释：后边的第一个boolean参数跟上边一样，第二个boolean类型的参数表示更新匹配到的所有记录
	语法：db.[collection].update({"key":"value"},{$set:{"key":"value"}},true|false，true|false)
	
	例子：更新所有name为“dozx”的数据
		db.csbn.update({"name":"dozx"},{$set:{"name":"ketty"}},false,true)


## 删除操作 ##


> 删除数据：  

	注释：删除数据与条件匹配到的数据	
	语法：db.[collection].remove({key:value})
	
	例子：删除所有name为“dozx”的数据
		db.[collection].remove({"name":"dozx"})


> 删除表：  
	注释：删除某个文档(类似于mysql中的表)
	语法：db.[collection].drop()

	例子：删除名为“csbn”的表
		db.csbn.drop()	

# MongoDB索引 #
## 索引 ##


> 查询索引  :

	注释：查询某个文档
	语法:db.[collection].getIndexes()
	例子：查找csbn中的索引
		db.csbn.getIndexes()


> 创建索引 ： 

	注释:创建一个索引。1为指定按升序创建索引，如果你想按降序来创建索引指定为-1即可。
	语法：db.[collection].ensureIndex({key:1|-1})
	例子:在csbn中以“name”为关键字创建一个按升序的索引
		db.csbn.ensureIndex({name:1})



> 使用多个字段创建索引:  

	注释：使用多个字段创建索引(复合索引)
	语法：db.[collection].ensureIndex({key1:1|-1,key2:1|-1})
	例子：在csbn中使用name,age两个字段创建索引
		db.csbn.ensureIndex({name:1,age:-1})

![img](https://gitee.com/mask616/images-bed/raw/master/typora-images/13caGnN.png)

> 实例:

	在后台创建索引：
	db.values.ensureIndex({open: 1, close: 1}, {background: true})

通过在创建索引时加background:true 的选项，让创建


> 多建索引：  

	例如db.[collection].insert({x:[1,2,3,4,5]})
	x即为一个多件索引。即某字段的值为一个数组的时候，为该字段创建一个索引即为多键索引

> 复合索引  :

	注释：需要根据多个条件查询时，需要复合索引.(1:表示按照按照升序创建索引。-1表示按照降序创建创建索引)
	语法：db.[collection].ensureIndex({key1: 1 | -1, key2: 1 | -1})
	例子：以x,y两个字段为索引创建复合索引
		db.csbn.ensureIndex({x:1,y:1})
> 过期索引  :

	释义：在一段时间后会过期的索引。在索引过期后，相应的数据会删除.
	应用场景：存储一些在一段时间之后会失效的数据，比如用户的登录信息，存储日志。 
	
	注释：expireAfterSeconds参数表示经过多长时间索引过期，单位为秒。
	语法：db.[collection].ensureIndex({key1:1|-1},{expireAfterSeconds:integer})
	例子：为csbn表以x字段为索引，过期时间设为10秒
		db.csbn.ensureIndex({x:1},{expireAfterSeconds:10})
	
	注意：1.存储在过期索引字段的值必须是指定的时间类型(必须是ISODate类型，或者是ISODate数组，否则不能被自动删除)
		 2.如果指定了ISODate数组，则按照最小的时间进行删除
		 3.过起索引不能是复合索引。
		 4.删除时间不是精确
		   说明：删除过程是由后台程序每60秒跑一次，而且删除也需要一些时间，所以存在误差

![img](https://gitee.com/mask616/images-bed/raw/master/typora-images/lZ7pLmQ.png)


> 图中标注注释：    

标注一：创建一个过期索引，过期时间为30秒。  
标注二：插入一条数据，key为“x”，value为“当前时间”(使用new Date()函数)。  
标注三：刚插入后查询数据，可以查询出来。  
标注四：过30秒后再次查询，刚插入的数据已经被自动删除。  


菜鸟一枚，大神勿喷。


