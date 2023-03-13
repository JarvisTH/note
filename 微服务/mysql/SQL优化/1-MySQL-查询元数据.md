元数据指数据的数据，如表名、列名、列类型、索引名等表的各种属性名称。MySQL使用information_schema数据库记录元数据信息，是一个虚拟数据库，物理上并不存在相关的目录和文件；库里的show tables显示的各种“表”是视图。

常用的视图：
- SCHEMATA：提供当前mysql实例中所有数据库的信息，即show databases的结果。
- TABLES：提供关于数据库中表的信息（包括视图），详细描述了某个表属于哪个schema、表类型、表引擎、创建时间等信息，即show tables from schemaname结果。
- COLUMNS：提供表中的列信息，详细描述某张表的所有列及每个列的信息，及show columns from schemaname.tablename结果。
- STATISTICS：提供关于表索引的信息，及show index from schemaname.tablename结果。
