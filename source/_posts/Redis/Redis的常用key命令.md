---
title: Redis学习（二）：Redis的常用key命令
date: 2020-02-27T13:30
author: codezhw

summary: 
categories: Redis
tags: 
- Redis
- Linux
---







# Redis的常用key命令



- **常用的key命令:**



**DEL key**

> 该命令用于在 key 存在时删除 key。 成功返回1，失败返回0

**EXISTS  key**

> 判断key是否存在  存在返回1，不存在返回0

**EXPIRE key seconds**

> 给key设置生存时间 单位：秒

**TTL key**

> 返回key 剩余的生存时间  

**MOVE key db**

> 将当前数据库的 key 移动到给定的数据库 db 当中。 

TYPE key

> 返回key的数据类型

**RENAME key newkey**

> 将key重命名为newkey

# Redis数据类型



String 类型 Java String