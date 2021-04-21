# MongoDB学习记录(四) #

## 索引属性 ##
> 为索引指定索引名：

	语法：db.[collection].ensureIndex({key1:1|-1,key2:1|-1},{name:"name"})
	例子：在csbn表中创建一个名为“normal_index”的索引
		db.csbn.ensureIndex({name:1,age:1},{name:"normal_index"})

> 创建唯一(unique)索引：

	释义：创建唯一索引。索引的关键字，不允许有重复的数据存在。比如以“nage”，“age”  
		为索引关键字。则不能存在name跟age一样的两条数据。
	语法：db.[collection].ensureIndex({key1:1|-1,key2:1|-1},{unique:true})
	例子：在csbn表中创建以name,age字段为索引，“normal_index”为索引名的唯一索引
		db.csbn.ensureIndex({name:1,age:1},{name:"normal_index",unique:true})
	
	注意:比如csbn表中已经存在name为“aa  bb  cc”，age为“0”的document，如果再插入一条  
		name为“aa  bb  cc”，age为“0”的document，则会报以下错误：
		
		> db.csbn.insert({name:"aa bb cc",age:0})
			WriteResult({
	    	"nInserted" : 0,
	    	"writeError" : {
	        "code" : 11000,
	        "errmsg" : "E11000 duplicate key error collection: first.csbn index: normal_index
			up key: { : \"aa bb cc\", : 0.0 }"
	    	}
		})

**查询语句补充**  


> 查询存在某字段的文档：	

	注释：MongoDB中时候不存在表结构的，没有固定的字段。当我们想查找存在某字段的文档时，需要用到“$exists”操作符。当$exists为“true”时，则表示查找存在某字段的值为false时表示不存在查找不存在某字段的文档
	语法：db.[collection].find({name:{$exists:true|false}})
	例子：查找csbn表中存在“desc”字段的集合
		db.csbn.find({desc:{$exists:true}})


> 稀疏度  

	注释：MongoDB中时候不存在表结构的，没有固定的字段。当表中存在索引，我们插入文档时，MongoDB会默认为这条文档添加索引。但是我们插入的文档可能不存在索引的那个字段。(例如csbn中存在“name”字段的索引，但是当我们使用db.csbn.insert({age:11}),这时MongoDB仍会为这个文档创建索引，但是实际上我们并不用为这个文档创建索引。)
	
	语法：db.[collection].ensureIndex({},{sparse:true|false})
	例子：为csbn表中的“name”字段创建稀疏索引
	db.csbn.ensureIndex({name:1},{sparse:true})
	
	创建索引后，强制使用索引查找不存在name字段的数据(hint()表示强制使用索引，里边的参数为索引名)
	db.csbn.find({name:{$exists:true}}).hint("name_1")
	这条语句会过滤不存在“name”字段的文档


## 地理位置索引 ##

> 创建地理位置索引：  

	注释：地理位置索引的参数共有两种，一种为“2d”，是基于平面的地理位置；  
		一种为“2dsphere”，是基于球面的地理位置索引。
	语法：db.[collection].ensureIndex({"key":"2d"})
	例子：在location表中为“place”字段创建地理位置索引
		db.location.ensureIndex({"x":"2d"})  
	
	插入数据的语法：地理位置的字段插入的时候为一个数组，表示经纬度
	 	位置表示方式：经纬度[经度，纬度]
	 	取值范围：经度[-180，180] 纬度[-90,90]
	 	例子：db.location.insert({x:["100","80"]})
	 	
	查找离[1,1]近的点(mongodb默认会查询出离查询的点最近的100个点)：
	 	db.location.find({x:{$near:[1,1]}})
	 		
	如果想指定返回结果的个数可以使用limit函数：	
	 	db.location.find({x:{$near:[1,1]}}).limit(5)
	 		
	查询最远距离内的点(使用$maxDistance限制最远距离为100)：
		db.location.find({x:{$near:[1,1],$maxDistance:100}})


> 查询地理位置索引某个形状内的点：

	查询矩形内的点：
		
		注释：使用{$geoWithin:{$box:[x1,y1],[x2,y2]}}查找矩形内的点.  
			[x1,y1]，[x2,y2]表示构成矩形的两个点
		语法：db.[collection].find({"key":{$geoWithin:{$box:[[x1,y1],[x2,y2]]}}})
		例子：查找[1,1][100,100]两点组成的矩形内的点
			 db.location.find({x:{$geoWithin:{$box:[[1,1],[100,100]]}}})


	查找圆形内的点：
		
		注释：使用{$geoWithin:{$center:[x1,y1],[r]}查找圆形内的点。  
			[x1,y1]表示圆心，[r]表示半径。
		语法：db.[collection].find({"key":{$geoWithin:{$center:[x1,y1],r}}})
		例子：查找以[1,1]为圆心，100为半径的组成的圆内的点。
			 db.location.find({x:{$geoWithin:{$center:[[1,1],100]}}})

> 2d索引的另一种查询方式：  

	geoNear查询：
	语法：
	db,runCommand({
		geoNear:<collection>,
		near:[x,y]
		maxDistance:
		num:
	})
	注：num表示想要返回的数据的条数
	
	例子：在location表中查询一条离[1,1]点最近的数据，并且限制最远的距离为100。
	
		db.runCommand({
			geoNear:"location",
			near:[1,1],
			maxDistance:100,
			num:1
		})
	返回结果如下图所示:
		results:表示查询到的结果
			dis：表示与[1,1]点的距离
			obj：表示返回的数据
		stats：表示一些系统信息
			nscanned：表示查询了多少条数据
			avgDistance:表示平均距离
			maxDistance:表示最远的那个点的距离
			time:表示查询花费的时间
![img](https://gitee.com/mask616/images-bed/raw/master/typora-images/OWkE2q2.jpg)

**菜鸟一枚，大神勿喷。**
