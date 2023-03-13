**1.正则表达式**
MySQL利用regexp命令提供正则表达式功能，匹配时区分大小写。
![](https://upload-images.jianshu.io/upload_images/9449419-0998ce11b3cfc1f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**2.巧用rand（）提取随机行**
原理是order by rand（）能够把数据随机排序。

**3.利用group by的with rollup子句**
可以检索出更多的分组聚合信息。with rollup反映的是一种olap思想，就是一个group by语句执行完成后可以满足用户想要得到的任何一个分组以及分组组合的聚合信息值。

**4.用bit group functions做统计**
共同使用group by和bit_and、bit_or函数完成统计。这两个函数一般做数值间的逻辑位运算。

**6.数据库名、表名大小写问题**
操作系统的大小写敏感性决定了数据库名、表名大小写敏感性。列、索引、存储子程序和触发器在任何平台上对大小写不敏感。总是用小写创建并引用数据库名和表名。

**7.使用外键需要注意的问题**
在MySQL中，innodb存储引擎对外部关键字约束条件检查
