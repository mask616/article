# MongoDB学习记录(三) #

## 索引(续) ##
####   补充  ####
> 删除索引：  

	语法：db.[collection].dropIndex("index name")
	例子：删除csbn表中，索引名为"name_1"的索引
	
	step1:查看索引csbn表中的索引： 
		db.csbn.getIndexes()
	step2:删除索引名为"name_1"：
		db.csbn.dropIndex("name_1")		


​												
### 全文索引 ###

> 创建全文索引:  

	注释：以“name”字段为索引创建全文索引,"text"为固定值，不可改变。
	语法：db.[collection].ensure({"key":"text"})
	例子：以字段"name"为索引创建全文索引
		db.csbn.ensureIndex({name:"text"})

> 使用全文索引查询：  

	下边的图片过程为：
		step1.查看csbn中的数据
		step2.查看csbn中的索引
		step3.创建一个全文索引
		step4.查看创建全文索引后的索引
		step5.通过全文索引的方式查询数据


​		
*step1:查看csbn中的数据*

![img](https://gitee.com/mask616/images-bed/raw/master/typora-images/aGDe2BS.png)

*step2:查看csbn中的索引*

![img](https://gitee.com/mask616/images-bed/raw/master/typora-images/ekhbFEH.png)

*step3:创建一个全文索引*

![img](https://gitee.com/mask616/images-bed/raw/master/typora-images/OE2Ku3c.png)

*step4:查看创建全文索引后的索引*

![img](https://gitee.com/mask616/images-bed/raw/master/typora-images/CSvuQtl.png)

*step5:通过全文索引的方式查询数据*

![img](https://gitee.com/mask616/images-bed/raw/master/typora-images/aQwzp05.png)

> MongoDB中几种全文索引查询的方式：  

	MongoDB全文索引查询的语法：
		(一)：
			注释:查找关键字“word”
			语法:db.[collection].find($text:{$search:"word"})
			例子:在csbn表中搜索关键字“sorry”
				db.csbn.find($text:{$search:"sorry"})
		(二):
			注释:搜索“word1”，“word2”关键字，“word1”，“word2”之间的空格表示或
			语法:db.[collection].find($text:{$search:"word1 word2"})
			例子:查询csbn表中存在关键字“so”或“sorry”的数据
				db.csbn.find($text:{$search:"so sorry"})
		(三):
			注释:搜索存在关键字“word1”或“word2”,但不包含“word3”的数据
			语法:db.[collection].find($text:{$search:"word1 word2 -word3"})
			例子:查询csbn表中存在关键字“so”或“sorry”，但包含“end”的数据
				db.csbn.find($text:{$search:"so sorry -end"})
		(四):
			注释:查询包含“word1”和“word2”和“word3”。用引号表示 “与”
			语法:db.[collection].find($text:{$search:"\"word\" \"word2\" \"word3\""})
			例子:查询csbn表中包含“best”和“friend”和“forever”的数据
				db.[collection].find($text:{$search:"\"best\" \"friend\" \"forever\""})
> 相似度全文检索  

	释义：“相似度”顾名思义会检索出与检索条件相似的结果。类似与我们在百度  
		  搜索的时候，会搜索出好多与查询条件相似的结果。使用  
		  {score:{$meta:"scoreText"}}表示相似度查询
	语法：db.[collection].find({$text:{$search:"word1 word2"}},{score:{$meta:"textScore"}})
	例子：查询与"aa"或"bb"相似的数据，查询出的数据的score即为相似度
		db.csbn.find({$text:{$search:"aa bb"}},{score:{$meta:"textScore"}})
![img](https://gitee.com/mask616/images-bed/raw/master/typora-images/r9hzoSC.png)

**按相似度排序查询：**
		

	语法：db.[collection].find({$text:{$search:"word1 word2"}},{score:{$mate:"textScore"}}).sort({score:{$mate:"textScore"}})
	例子：按相似度字段为关键词降序
		db.csbn.find({$text:{$search:"aa xx"}},{score:{$meta:"textScore"}}).sort({score:{$meta:"textScore"}})
![img](https://gitee.com/mask616/images-bed/raw/master/typora-images/sWYeh1M.png)


**菜鸟一枚，大神勿喷。**
