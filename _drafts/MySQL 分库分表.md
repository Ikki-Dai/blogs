# MySQL 分库分表

## 主键设计

(树(MySQL), Hash(Redis), 列存储(LSM))

- 聚簇索引
  - 数据存储在主键索引中
  - 数据按主键顺序存储

- 二级索引

  - 除主键以外的索引
  - 叶子节点中主键值
  - 一次查询需要走两遍索引
  - 回表：二级索引指向主键索引
  - 主键大小会影响所有索引大小

- 联合索引
  - 多个字段组成
  - 最左匹配原则
  - 一个索引只创建一颗树
  - 按第一列排序，第一列相同按第二列排序

  BIGINT 层 10 亿条数据

  16KB/(8B (key)+ 8B(指针)) = 1K

  10^3 * 10^3 * 10^3 = 10 亿

  32 字节 64