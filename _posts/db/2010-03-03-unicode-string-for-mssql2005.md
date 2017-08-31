---
title: Unicode string in SELECT query for MS SQL 2005
---

## Unicode string in SELECT query for MS SQL 2005

{{ page.date | date_to_string }} 同事在工作上遇到一個莫名的問題....

問題是出自於以下的SQL

`SELECT * FROM table_A WHERE column_A = '深水埗區'`

在Table_A的Column_A中有很多用UTF-8來儲存的文字，當然當中包含了'深水埗區'，但以上這SQL沒有回傳任何Data！Google之下才知在SQL Server 2005中處理Unicode字串常數時，必需為所有的Unicode字串加上prefix "N"。如果不加“N”的話，SQL Server就會把Unicode字串轉為Default Code Page。

所以要把以上的SQL statement 改為:

`SELECT * FROM table_A WHERE column_A = N'深水埗區'`

詳細的說明可看這:
[http://support.microsoft.com/kb/239530](http://support.microsoft.com/kb/239530)
