# sparql-to-gremlin 文档整理

原文：[Gremlinator: An effort towards converting SPARQL queries to Gremlin Graph Pattern Matching Traversals](https://github.com/LITMUS-Benchmark-Suite/sparql-to-gremlin)

该篇为上述文档内容整理。<!--非全文翻译-->



### Gremlinator文档

本文档提供了SPARQL-Gremlin编译器（也称为Gremlinator）的参考文档，该编译器**用于将SPARQL查询转换为Gremlin遍历**。 它基于Apache Jena SPARQL处理器ARQ，该处理器提供对SPARQL查询的语法树的访问。

当前版本的SPARQL-Gremlin仅使用Apache Jena提供的功能的子集。 以下示例显示了每个已实现的功能<!--该篇文档版本较早，现在已更新到3.4.6-->



### 介绍

 通过将SPARQL查询转换为Gremlin模式匹配遍历来实现Gremlin查询语言自动支持在Graph系统上执行SPARQL查询。



### 样例演示

1. Gremlinator演示- [演示1](http://gremlinator.iai.uni-bonn.de:8080/Demo/)和[演示2](http://195.201.31.31:8080/Demo/)<!--演示2可能打不开,参考演示1即可-->
2. 有关如何使用演示的简短视频教程- [视频教程](https://www.youtube.com/watch?v=Z0ETx2IBamw)



样例说明（演示1）：

​	其中有两个数据集可选：Northwind 和 BSBM.

* Northwind有8张表，涉及客户、商品、订单       [介绍](https://blog.csdn.net/pan15125284/article/details/91864509)

![img](http://gremlinator.iai.uni-bonn.de:8080/Demo/images/northwind-schema.png)

* BSBM

![img](http://gremlinator.iai.uni-bonn.de:8080/Demo/images/bsbm-schema.png)





查询设计说明：

![img](http://gremlinator.iai.uni-bonn.de:8080/Demo/images/Query_Design_Description.png)



SPARQL可修改：

```
select ?b
where {
                ?a v:label "product" .
                ?a v:name ?b .
}  
```

转换成Gremlin遍历：

```
[GraphStep(vertex,[]), MatchStep(AND,[[MatchStartStep(a), HasStep([~label.eq(product)]), MatchEndStep], [MatchStartStep(a), PropertiesStep([name],value), MatchEndStep(b)]]), SelectOneStep(b)] 
```

返回结果：

```
[Chai, Chang, Aniseed Syrup, Chef Anton's Cajun Seasoning, Chef Anton's Gumbo Mix, Grandma's Boysenberry Spread, Uncle Bob's Organic Dried Pears, Northwoods Cranberry Sauce, Mishi Kobe Niku, Ikura, Queso Cabrales, Queso Manchego La Pastora, Konbu, Tofu, Genen Shouyu, Pavlova, Alice Mutton, Carnarvon Tigers, Teatime Chocolate Biscuits, Sir Rodney's Marmalade, Sir Rodney's Scones, Gustaf's Knäckebröd, Tunnbröd, Guaraná Fantástica, NuNuCa Nuß-Nougat-Creme, Gumbär Gummibärchen, Schoggi Schokolade, Rössle Sauerkraut, Thüringer Rostbratwurst, Nord-Ost Matjeshering, Gorgonzola Telino, Mascarpone Fabioli, Geitost, Sasquatch Ale, Steeleye Stout, Inlagd Sill, Gravad lax, Côte de Blaye, Chartreuse verte, Boston Crab Meat, Jack's New England Clam Chowder, Singaporean Hokkien Fried Mee, Ipoh Coffee, Gula Malacca, Rogede sild, Spegesild, Zaanse koeken, Chocolade, Maxilaku, Valkoinen suklaa, Manjimup Dried Apples, Filo Mix, Perth Pasties, Tourtière, Pâté chinois, Gnocchi di nonna Alice, Ravioli Angelo, Escargots de Bourgogne, Raclette Courdavault, Camembert Pierrot, Sirop d'érable, Tarte au sucre, Vegie-spread, Wimmers gute Semmelknödel, Louisiana Fiery Hot Pepper Sauce, Louisiana Hot Spiced Okra, Laughing Lumberjack Lager, Scottish Longbreads, Gudbrandsdalsost, Outback Lager, Flotemysost, Mozzarella di Giovanni, Röd Kaviar, Longlife Tofu, Rhönbräu Klosterbier, Lakkalikööri, Original Frankfurter grüne Soße] 
```

遍历简介：

![image-20200324092831477](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200324092831477.png)



### 支持的SPARQL查询类型

* Union
* Optional
* Order-By
* Group-By
* STAR-shaped or neighbourhood queries
* Query modifiers, such as:
  * Filter with restrictions
  * Count
  * LIMIT
  * OFFSET



### 限制

Gremlinator的当前实现（即SPARQL-Gremlin）不支持以下功能：

* 当前不覆盖谓词位置中具有变量的SPARQL查询：

```
SELECT ?x WHERE { ?x ?y ?z . }
```

* 仅当SPARQL查询的联合运算符两边具有相同数量的模式时，才能生成具有不平衡模式的SPARQL Union查询，即gremlin联合遍历。例如，以下SPARQL查询无法使用Gremlinator进行映射，因为在不同数量的图形模式之间执行了并集。

```
SELECT * WHERE {
  {?person e:created ?software .
  ?person v:name "daniel" .}
  UNION
  {?software v:lang "java" .}
}
```



### 用法 <!-- 更新为3.4.6版本-->

1.下载插件（该步骤时间可能较长，请耐心等待）

```
gremlin> :install org.apache.tinkerpop sparql-gremlin 3.4.6
==>Loaded: [org.apache.tinkerpop, sparql-gremlin, 3.4.6]
```

2.激活

```
gremlin> :plugin use tinkerpop.sparql
==>tinkerpop.sparql activated
```

3.使用实例

```
gremlin> graph = TinkerFactory.createModern()
==>tinkergraph[vertices:6 edges:6]
gremlin> g = graph.traversal(SparqlTraversalSource)
==>sparqltraversalsource[tinkergraph[vertices:6 edges:6], standard]
gremlin> g.sparql("""SELECT ?name ?age
......1>             WHERE { ?person v:name ?name . ?person v:age ?age }
......2>             ORDER BY ASC(?age)""")
==>[name:vadas,age:27]
==>[name:marko,age:29]
==>[name:josh,age:32]
==>[name:peter,age:35]
```



### SPARQL可用查询实例

* Select All

查询图中所有节点。

```
SELECT * WHERE { }
```

* Match Constant Values

查找所有标记为人的节点。

```
SELECT * WHERE {  ?person v:label "person" .
}
```

* Select Specific Elements

查询每个人的姓名和年龄。

```
SELECT ?name ?age
WHERE {
  ?person v:label "person" .
  ?person v:name ?name .
  ?person v:age ?age .
}
```

* Pattern Matching

查询创建项目的人的姓名和年龄。

```
SELECT ?name ?age
WHERE {
  ?person v:label "person" .
  ?person v:name ?name .
  ?person v:age ?age .
  ?person e:created ?project .
}
```

* Filtering

查询年龄大于30岁的人的姓名和年龄.

```
SELECT ?name ?age
WHERE {
  ?person v:label "person" .
  ?person v:name ?name .
  ?person v:age ?age .
  ?person e:created ?project .
    FILTER (?age > 30)
}
```

* Deduplication

查询创建项目且年龄大于30岁的人的姓名（去重）。

```
SELECT DISTINCT ?name
WHERE {
  ?person v:label "person" .
  ?person e:created ?project .
  ?project v:name ?name .
    FILTER (?age > 30)
}
```

* Multiple Filters

查询创建java项目且年龄大于30岁的人的姓名（去重）。

```
SELECT DISTINCT ?name
WHERE {
  ?person v:label "person" .
  ?person v:age ?age .
  ?person e:created ?project .
  ?project v:name ?name .
  ?project v:lang ?lang .
    FILTER (?age > 30 && ?lang == "java")
}
```

* Pattern Filter(s)

筛选创建项目的所有人的另一种方法。

```
SELECT ?name
WHERE {
  ?person v:label "person" .
  ?person v:name ?name .
    FILTER EXISTS { ?person e:created ?project }
}
```

筛选未创建项目的人。

```
SELECT ?name
WHERE {
  ?person v:label "person" .
  ?person v:name ?name .
    FILTER NOT EXISTS { ?person e:created ?project }
}
```

* Meta-Property Access

访问图元素的元属性。

```
SELECT ?name ?startTime
WHERE {
  ?person v:name "daniel" .
  ?person p:location ?location .
  ?location v:value ?name .
  ?location v:startTime ?startTime
}
```

* Union

查询使用Java开发发软件的所有人。

```
SELECT * WHERE {
  {?person e:created ?software .}
  UNION
  {?software v:lang "java" .}
}
```

* Optional

查询使用Java或python开发软件的人员的姓名。

```
SELECT ?person WHERE {
  ?person v:label "person" .
  ?person e:created ?software .
  ?software v:lang "java" .
  OPTIONAL {?software v:lang "python" . }
}
```

* Order By

按年龄给人员排序。

```
SELECT * WHERE {
  ?person v:label "person" .
  ?person v:age ?age .
} ORDER BY (?age)
```

* Group By

根据年龄将人员分组。

```
SELECT * WHERE {
  ?person v:label "person" .
  ?person v:age ?age .
} GROUP BY (?age)
```

* Mixed/complex/aggregation-based queries

计算30岁以下人员创建的项目数，并按年龄分组。 返回前两个。

```
SELECT COUNT(?project) WHERE {
  ?person v:label "person" .
  ?person v:age ?age . FILTER (?age < 30)
  ?person e:created ?project .
} GROUP BY (?age) LIMIT 2
```

* STAR-shaped queries

星形查询是形成或遵循星形执行计划的查询。 这些在图形遍历方面可以看作是路径查询或邻域查询。 例如，获取有关特定“个人”或“软件”的所有信息。

```
SELECT ?age ?software ?name ?location ?startTime WHERE {
  ?person v:name "daniel" .
  ?person v:age ?age .
  ?person e:created ?software .
  ?person p:location ?location .
  ?location v:value ?name .
  ?location v:startTime ?startTime
}
```

