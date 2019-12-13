---
title: Oracle去除特殊字符
date: 2019-12-13 15:38:22
categories: Oracle
tags: [SQL, 特殊字符]
---
在Oracle数据库中，有时字段中会存入一些特殊字符（设计不当或者程序BUG），例如制表符、换行符以及回车符等，如果包含特殊字符是看不出来的，但是在进行字段关联的时候就会查不出数据。
特殊字符ASCII码定义：
{% blockquote %}
制表符 chr(9) 
换行符 chr(10)
回车符 chr(13)
{% endblockquote %}

```SQL
update table set field=replace(field,chr(9), '') where instr(field, chr(9)) > 0
```

其他特殊字符也可以使用ascii函数查出ASCII码然后进行替换
```SQL
select ascii('?') from dual;
```