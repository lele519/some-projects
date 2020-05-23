# 数据库
## python内置的sqlite3模块
+ python内置sqlite3模块专门用来让我们创建内存数据库。
	+ 使我们不用下载安装专门的数据库软件。
	 + 因本章是关于在python中与数据库进行交互，所以主要介绍了通用的CRUD(create\read\update\delete)操作

### 使用python如何在数据库中执行命令，输入输出
```
#!/usr/bin/env python3
import sqlite3
###Create an in-memory SQLite3 database
###Create a table called sales with four attributes
con = sqlite3.connect(':memory:')#为了使用该模块，我们必须先创建一个代表数据库的连接对象
###此处用con代表数据库。
###专用名称【':memory:'】：在内存创建了一个数据库。
###如果你想让数据库持久话，则需要另外的字符串，eg：'my_database.db'或'路径\my_database.db'。这样数据库对象就会持久保存在当前目录上
query = """CREATE TABLE sales
			(customer VARCHAR(20), 
			 product VARCHAR(40),
			 amount FLOAT,
			 date DATE);"""
###用的"""来创建字符
###query这边是一个sql命令，创建了TABLE sales
con.execute(query)##用execute()：执行包含在变量query中的sql命令，此处 在con里面创建了TABLE
con.commit()##commit()将修改提交（保存）到数据库
###当你对数据库进行修改时，必须用commit()来保存，不然就不会保存到数据库
####Insert a few rows of data into the table
data = [('Richard Lucas', 'Notepad', 2.50, '2014-01-02'),
		('Jenny Kim', 'Binder', 4.15, '2014-01-15'),
		('Svetlana Crow', 'Printer', 155.75, '2014-02-03'),
		('Stephen Randolph', 'Computer', 679.40, '2014-02-20')]##创建data
statement = "INSERT INTO sales VALUES(?, ?, ?, ?)"##建立一个变量statement，使用insert命令，'？'为占位符
con.executemany(statement, data)##executemany():为data中的每个数据元组执行变量statement中的sql命令
con.commit()
####Query the sales table
cursor = con.execute("SELECT * FROM sales")##此处使用连接对象的execute()运行一条sql命令，并将结果赋值给cursor
rows = cursor.fetchall()##通常使用fetchall()取出（返回）结果集中的所有行
#####Count the number of rows in the output
row_counter = 0
for row in rows:
	print(row)
	row_counter += 1
print('Number of rows: {}'.format(row_counter))
```
### 向表中插入新数据
#### 此处是从csv格式的输入文件将数据批量地加载到数据库的表中
```
#!/usr/bin/env python3
import csv
import sqlite3
import sys
###Path to and name of a CSV input file
input_file = sys.argv[1]
###Create an in-memory SQLite3 database
###Create a table called Suppliers with five attributes
con = sqlite3.connect(r'C:\Users\86152\Desktop\database\Suppliers.db')###记得写r，确保python不处理字符串之间的转义符
###或者将\换成 \\ 或者 /，都不会出现error
###创建本地连接,并将其保存起来
c = con.cursor()
create_table = """CREATE TABLE IF NOT EXISTS Suppliers
				(Supplier_Name VARCHAR(20), 
				Invoice_Number VARCHAR(20),
				Part_Number VARCHAR(20),
				Cost FLOAT,
				Purchase_Date DATE);"""
c.execute(create_table)
con.commit()
###Read the CSV file
###Insert the data into the Suppliers table
file_reader = csv.reader(open(input_file, 'r'), delimiter=',')###'r'表read
###打开文件，读取内容，" delimiter=',' :表','为默认分隔符。#如果输入文件是用逗号分隔则不用指定，但是此处防止输入/输出文件具有不同的分割符
header = next(file_reader, None)
for row in file_reader:
	data = []
	for column_index in range(len(header)):
		data.append(row[column_index])
	print(data)##此处是用来调试脚本。这个print是在外部for循环下，而不是内部for循环下。
	##如果你调试完毕，且代码正常运行，完全可以删掉。或者注释掉。以防止过多输出在屏幕上
	c.execute("INSERT INTO Suppliers VALUES (?, ?, ?, ?, ?);", data)
con.commit()
###Query the Suppliers table
output = c.execute("SELECT * FROM Suppliers")
rows = output.fetchall()
for row in rows:
	output = []
	for column_index in range(len(row)):
		output.append(str(row[column_index]))
	print(output)
```
### 更新表中的数据

```
#!/usr/bin/env python3
import csv
import sqlite3
import sys
###Path to and name of a CSV input file
input_file = sys.argv[1]
###Create an in-memory SQLite3 database
###Create a table called sales with four attributes
con = sqlite3.connect(r'C:\Users\86152\Desktop\database\3output.db')
query = """CREATE TABLE IF NOT EXISTS sales
			(customer VARCHAR(20), 
				product VARCHAR(40),
				amount FLOAT,
				date DATE);"""
con.execute(query)
con.commit()
###Insert a few rows of data into the table
data = [('Richard Lucas', 'Notepad', 2.50, '2014-01-02'),
		('Jenny Kim', 'Binder', 4.15, '2014-01-15'),
		('Svetlana Crow', 'Printer', 155.75, '2014-02-03'),
		('Stephen Randolph', 'Computer', 679.40, '2014-02-20')]
for tuple in data:
	print(tuple)
statement = "INSERT INTO sales VALUES(?, ?, ?, ?)"
con.executemany(statement, data)
con.commit()
###Read the CSV file and update the specific rows
file_reader = csv.reader(open(input_file, 'r'), delimiter=',')
header = next(file_reader, None)
for row in file_reader:
	data = []
	for column_index in range(len(header)):
		data.append(row[column_index])
	print(data)
	###接下来就是见证update的时刻
	###在update语句中，必须指定想要更新哪一条记录和哪一个列属性。
	con.execute("UPDATE sales SET amount=?, date=? WHERE customer=?;", data)###此例中，查询中的属性是mount, date，customer。因此csv输入文件中的列也应是mount, date，customer顺序
con.commit()
###Query the sales table
cursor = con.execute("SELECT * FROM sales")
rows = cursor.fetchall()
for row in rows:
	output = []
	for column_index in range(len(row)):
		output.append(str(row[column_index]))
	print(output)
```
## MySQL数据库
### MySQL简介
#### 创建一个数据库
- CREATE DATABASE 数据库名（英文为佳）
	- SHOW DATABASE
	 - 用以检查
- 使用数据库
	- USE 数据库名
- 建表
	- CREATE TABLE IF NOT EXISTS 表名; （不推荐）
		-如果指定了if not exists语句来创建表,如果表存在,也不会报错。创建表的语句不会验证要创建的表与已经存在的表的结构是否一致,只要名字相同就不允许创建. 
	- CREATE TABLE 表名（ 列一名 varchard（20）， 列二名  varchard（10），列三名  FLOAT，列四名（假如是date） DATE);
- 表格属性检查
	- DESCRIBE 表名；
- 新用户创建
	- CREATE USER '用户名'@'localhost' IDENTIFIED BY '你制定的密码'；
- 给新用户授权
	- GRANT ALL PRIBILEGES ON 某数据库名.* TO '用户名'@'localhost';

#### 向表中插入记录
- 该例是将csv文件记录转入数据库
```
#!/usr/bin/env python3
import csv
import pymysql as MySQLdb #书上import了MySQLdb,但是实际的包是pymysql，为了节省时间，没有把所有的包名改动，用了as
import sys
from datetime import *
##Path to and name of a CSV input file
input_file = sys.argv[1]
##Connect to a MySQL database
#此处要用自己电脑的各种密码host='localhost'为47.114.72.175（远程服务器）, port=5002, db='之前建的db名', user='新建的用户名', passwd='自己的密码'
##这五个参数创立了与my_suppliers数据库的本地连接
con = MySQLdb.connect(host='localhost', port=5002, db='my_suppliers', user='lele', passwd='1234')
c = con.cursor()##创建了光标，用来在my_suppliers数据库中对suppliers表执行SQL语句，并将修改提交到数据库
##对于host我们扯个皮，如果mysql在本机上什么都好商量，但是我的是在他人的服务器上，我们就要把host = '服务器所在的机器的主机名'，由于，可能，人家做了防火墙，所以我现在访问不进去，但是还是可以尝试的。
##port是Mysql服务器的TCP/IP连接端口号。需要是有效端口号。
		##如果又是不在本机工作，需要确保是有效的端口号。
##Read the CSV file
##Insert the data into the Suppliers table
file_reader = csv.reader(open(input_file, 'r'), delimiter=',')
header = next(file_reader)
for row in file_reader:
	data = []
	for column_index in range(len(header)):
		if column_index < 4:#处理除最后一列（除日期）以外的所有列的数据，并加入data
			data.append(str(row[column_index]).lstrip('$').replace(',', '').strip())
		else:
			a_date = datetime.date(datetime.strptime(str(row[column_index]), '%m/%d/%y'))##先洗成月日年（两位年）
			##%Y: year is 2016; %y: year is 15
			a_date = a_date.strftime('%Y-%m-%d')#再洗成年（四位年）月日
			data.append(a_date)
	print(data)
	c.execute("""INSERT INTO Suppliers VALUES (%s, %s, %s, %s, %s);""", data)
con.commit()##数据库必须确认呢，不然告辞
# Query the Suppliers table
c.execute("SELECT * FROM Suppliers")#使用 execute() 方法执行 SQL,并将结果赋值给cursor，此处是：从Suppliers表中选择所有数据
rows = c.fetchall()##通常使用fetchall()取出（返回）结果集中的所有行，此处：将输出中的所有行读入变量
for row in rows:
	row_list_output = []
	for column_index in range(len(row)):
		row_list_output.append(str(row[column_index]))
	print(row_list_output)
```
#### 查询一个表并将输出写入csv文件
```
#!/usr/bin/env python3
import csv
import pymysql as MySQLdb #书上import了MySQLdb,但是实际的包是pymysql，为了节省时间，没有把所有的包名改动，用了as
import sys
###Path to and name of a CSV output file
output_file = sys.argv[1]
##Connect to a MySQL database
#此处要用自己电脑的各种密码host='localhost'为47.114.72.175（远程服务器）, port=5002, db='之前建的db名', user='新建的用户名', passwd='自己的密码'
##这五个参数创立了与my_suppliers数据库的本地连接
con = MySQLdb.connect(host='127.0.0.1', port=5002, db='my_suppliers', user='lele', passwd='1234')
c = con.cursor()##创建了光标，用来在my_suppliers数据库中对suppliers表执行SQL语句，并将修改提交到数据库
##对于host我们扯个皮，如果mysql在本机上什么都好商量，但是我的是在他人的服务器上，我们就要把host = '服务器所在的机器的主机名'，由于，可能，人家做了防火墙，所以我现在访问不进去，但是还是可以尝试的。
##port是Mysql服务器的TCP/IP连接端口号。需要是有效端口号。
		##如果又是不在本机工作，需要确保是有效的端口号。
###reate a file writer object and write the header row
filewriter = csv.writer(open(output_file, 'w', newline=''), delimiter=',')
header = ['Supplier Name','Invoice Number','Part Number','Cost','Purchase Date']#空手插title
filewriter.writerow(header)
###Query the Suppliers table and write the output to a CSV file
c.execute("""SELECT * 
		FROM Suppliers 
		WHERE Cost > 700.0;""")##execute()执行查询指令
rows = c.fetchall()##通常使用fetchall()取出（返回）结果集中的所有行
for row in rows:
	filewriter.writerow(row)
```
 
#### 更新表中记录
```
#!/usr/bin/env python3
import csv
import pymysql as MySQLdb #书上import了MySQLdb,但是实际的包是pymysql，为了节省时间，没有把所有的包名改动，用了as
import sys
###Path to and name of a CSV input file
input_file = sys.argv[1]
##Connect to a MySQL database
#此处要用自己电脑的各种密码host='localhost'为47.114.72.175（远程服务器）, port=5002, db='之前建的db名', user='新建的用户名', passwd='自己的密码'
##这五个参数创立了与my_suppliers数据库的本地连接
con = MySQLdb.connect(host='127.0.0.1', port=5002, db='my_suppliers', user='lele', passwd='1234')
c = con.cursor()##创建了光标，用来在my_suppliers数据库中对suppliers表执行SQL语句，并将修改提交到数据库
##对于host我们扯个皮，如果mysql在本机上什么都好商量，但是我的是在他人的服务器上，我们就要把host = '服务器所在的机器的主机名'，由于，可能，人家做了防火墙，所以我现在访问不进去，但是还是可以尝试的。
##port是Mysql服务器的TCP/IP连接端口号。需要是有效端口号。
		##如果又是不在本机工作，需要确保是有效的端口号。	
###Read the CSV file and update the specific rows
file_reader = csv.reader(open(input_file, 'r', newline=''), delimiter=',')
header = next(file_reader, None)
for row in file_reader:
	data = []
	for column_index in range(len(header)):
		data.append(str(row[column_index]).strip())
	print(data)
	c.execute("""UPDATE Suppliers SET Cost=%s, Purchase_Date=%s WHERE Supplier_Name=%s;""", data)##指令换成update
con.commit()
###Query the Suppliers table
c.execute("SELECT * FROM Suppliers")
rows = c.fetchall()
for row in rows:
	output = []
	for column_index in range(len(row)):
		output.append(str(row[column_index]))
	print(output)

```
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 