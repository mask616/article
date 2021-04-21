# MongoDB 学习记录(一)
注释：
	[db name]    : 数据库名  
	[collection] : 相当于关系型数据库中的表(table)

## 数据插入 ##
删除数据库：  

	1.切换到要删除的数据库
		use [db name]  
	2.删除数据库         
		db.dropDatabase()
查看所有的数据库：  

	show dbs;
写入数据：  

	1.切换到需要插入的数据库   
		use [db name]
	2.插入数据             
		db.[collection].insert({"key":"value","key":"value"})
		db.[collection].insert({"key":"value","key":["value1","value2"]})
查看表中的数据：  

	db.[collection].find()
删除集合(collection)：  

	db.[collection].drop()
条件查询：  

	1.按指定条件查询  
		db.[collection].find({key:value})
	2.类似于按页查询
		注释：skip(n) 跳过前n条数据；limit(n)查询n条数据；sort({key：value})按某个字段排序
		db.[collectin].find().skip(3).limit(2).sort(x:1)
使用for插入多条数据：  

	for(i = 0 ;i < 100 ; i++) db.[collection].insert(x:i)
查询文档中数据集合的数量：  

	db.[collection].find().count();

## 数据更新 ##

更新语句的格式：    

		db.collection.update(
			<query>,
			<update>,
			{
	 		upsert: <boolean>,
	 		multi: <boolean>,
	 		writeConcern: <document>
			}
		)
	
	参数说明：
		query : update的查询条件，类似sql update查询内where后面的。
		update : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
		upsert : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
		multi : 可选，MongoDB 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
		writeConcern :可选，抛出异常的级别。


更新按条件查询出来的数据：  

	注释：可更新数据中的key的值  
	db.[collection].update({修改前的key:修改前的value}，{修改后的key：修改后的value})
	eg:  db.[collection].update({x:1},{y:2}) (注：此更新会更新x为1的行数据的所有数据，效果如下图：)  

![img](https://gitee.com/mask616/images-bed/raw/master/typora-images/lnVE4FQ.png)

若要更新某个字段则需要使用修改器，MongoDB中使用‘$’做为修改器的符号  

![img](https://gitee.com/mask616/images-bed/raw/master/typora-images/I3FW30j.png)  

如图我们先添加了一条数据，然后修改。最后为修改后的效果，只修改了一个字段。可以看到姓名(name)由原来的 "dozx" 更新为了 "cxg"。

以上语句只会修改第一条发现的文档，如果你要修改多条相同的文档，则需要设置 multi 参数为 true。

	>db.csbn.update({"name":"dozx"},{$set:{"name":"cxg"}},{multi:true})

更多实例

只更新第一条记录：  

	db.csbn.update( { "count" : { $gt : 1 } } , { $set : { "test2" : "OK"} } );


全部更新：  

	db.csbn.update( { "count" : { $gt : 3 } } , { $set : { "test2" : "OK"} },false,true );

只添加第一条：  

	db.csbn.update( { "count" : { $gt : 4 } } , { $set : { "test5" : "OK"} },true,false );

全部添加加进去:  

	db.csbn.update( { "count" : { $gt : 5 } } , { $set : { "test5" : "OK"} },true,true );

全部更新：  

	db.csbn.update( { "count" : { $gt : 15 } } , { $inc : { "count" : 1} },false,true );

只更新第一条记录： 

	db.csbn.update( { "count" : { $gt : 10 } } , { $inc : { "count" : 1} },false,false );  

  




菜鸟一枚，大神勿喷
