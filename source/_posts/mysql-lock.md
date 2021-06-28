---
title: mysql锁浅析
date: 2021-06-18 18:52:57
tags: [mysql,技术]
---

<!-- more -->



| session A                                                 | session B                       | session C                                 |
| --------------------------------------------------------- | ------------------------------- | ----------------------------------------- |
| begin;<br/>select id from t where c=5 lock in share mode; |                                 |                                           |
|                                                           | update t set d=d+1 where id=5;' |                                           |
|                                                           |                                 | insert into values(7,7,7);<br/> (blocked) |

