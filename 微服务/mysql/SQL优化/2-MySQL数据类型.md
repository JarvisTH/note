1.**整数类型**
分为tinyint、smallint、mediumint、int和bigint，超出范围会发生“Out of range”提示。支持在类型名称后括号内指定显示宽度，不知道则是默认值，一般配合zerofill使用，在数字位数不够的空间用字符“0”填满。设置显示宽度不会影响实际插入的数据精度。

所有的整数类型都有一个可选属性UNSIGNED（无符号），若要保存非负数或者较大的上限值，可以使用这个，取值下限是0，上限是原值的2倍。

AUTO_INCREMENT：在产生唯一标识符或顺序值时使用，只属于整数类型。一般值从1开始，每行增加1。在插入一个NULL到AUTO_INCREMENT列时，MySQL插入一个比该列当前最大值大1的值。表中最多只能有一个自增列，且该列为not null，并定义为primary key或者unique键。

2.**小数**
浮点数和定点数。浮点数包括float、double，定点数只有decimal。定点数中MySQL内部以字符串形式存放，比浮点数精度高，适合表示货币等高精度数据。

都可以用类型名称加（M，D）方式表示，M表示位数，D表示小数位数。float和double在不指定精度时，默认按实际精度显示；decimal则默认整数位10，小数位0，超过精度值会报错。

3.**bit**
可以用来存放多位二进制数，范围从1~64，默认1位。对于位字段，select命令不会看到结果，用bin（）（显示二进制格式）或者hex（）（显示16进制格式）函数读取。

数据插入bit类型字段时，首先转换为二进制，若位数允许，将成功插入；若位数小于实际定义位数，则插入失败。

4.**日期时间类型**
- 表示年月日，常用DATE
- 表示年月日时分秒，常用DATETIME
- 表示时分秒，常用TIME
- 若需要经常插入或更新日期为当前系统时间，常用TIMESTAMP。TIMESTAMP字段只能有一列默认值为current_timestamp。特点与时区相关，插入时先转换为本地时区后存放，读取时转换为本地时区后显示。
与DATETIME区别：
1.TIMESTAMP支持时间范围小
2.表中第一个TIMESTAMP列字段设置系统时间；插入NULL，自动设置系统时间。
3.TIMESTAMP插入与查询受到时区影响。
- 只表示年份，用year，比DATE占用空间少。

5.**字符串类型**
![](https://upload-images.jianshu.io/upload_images/9449419-e3fd29f6c899ad21.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

char列在检索时删除尾部空格，varchar则保留空格；

binary与varbinary，类似char与varchar，不同的是包含二进制字符串而不包含非二进制字符串。

对于枚举类型，忽略大小写，存储时转为大写，对于插入值不在范围内时，会插入第一个值。只允许从值集合中选区单个值。

set类型与enum类似，主要区别是set类型一次可以选区多个成员。
![](https://upload-images.jianshu.io/upload_images/9449419-073dda24b7247e81.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

