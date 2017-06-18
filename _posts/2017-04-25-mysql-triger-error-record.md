---
layout: post
title: 记一次mysql触发器的错误经历
date:   2017-4-25 10:30:00 +0800
categories: Mysql 
tag: [mysql]
---

* content
{:toc}
 
## 前言

在工作中遇到一个需求，优惠券的记录的表，要求每生成一条优惠券记录，附带生成唯一的16个字符长度的号码，一开始想要mysql触发器实现，然而。。。

### 第一次尝试

表的结构
```
create table `card`(
    `id` int(11) not null primary key,
    `code` char(16) default null,
    -- 省略其他字段
)ENGINE=InnoDB auto_increment=0 charset=utf8;

```

新建触发器

```sql
use `database`;
delimiter $$
create trigger `card_insert_trigger` after insert
     on `card` for each row
     BEGIN
     update `card` set `code` = concat(left(concat(FROM_UNIXTIME(UNIX_TIMESTAMP(),'%Y%m%d'),floor(rand()*POWER(10,8))),16-length(`id`)),`id`)  where `code` is null or `code`='';
     END 
     $$
delimiter ;
```

简单说明：
- `delimiter`表示修改mysql的sql语句的分界符，默认`;`，但是触发器中的语句往往带有`；`会对新建触发器的sql语句造成语法识别错误，于是改为不常用的字符，例如`$$`
- 这一条触发器的意思是，当