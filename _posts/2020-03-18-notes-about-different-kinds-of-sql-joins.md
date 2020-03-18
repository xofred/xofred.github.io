---
title:  "Notes about Different Kinds of SQL Joins"
tags: sql mysql

---

*https://www.dofactory.com/sql/join*

![img](https://www.dofactory.com/Images/sql-joins.png)

The image above gives a pretty good concept about different kinds of joins of SQL.

But there is gotcha in MySQL, cause it has no FULL JOIN. We can only mimic it by first LEFT JOIN, then RIGHT JOIN, then UNION them both. It's kind of stupid but we have to live with it, as long as we use MySQL.

Everyone has his strength, MySQL too. Lately I discovered, it has a cool built-in feature called 'partition', which is great for large tables. I might write a blog about it later.

During the wrestling with SQL writing and discussing business logic with colleague. I found [a handy online tool](http://dpriver.com/pp/sqlformat.htm) which can beautify SQL instantly. Which is great, if you happen to deal with long and nasty SQL.

*p.s. This is my first blog in English. And I intend to keep it that way. Why? Because I live in a country which government forbid people to say what they want to say. I am prisoner of my own country.*
