# csv 文件
+ 不要再试图尝试print filewriter了，print那个鬼是给文件写内容啊！！！！！放弃吧！！！！！
## 读写文件part1
### 基础python 不使用csv模块

```
#!/usr/bin/env python3
import sys
input_file = sys.argv[1]
output_file = sys.argv[2]
#newline = ' ' : csv文件的每一行都读取为一个由字符串组成的列表。除非指定了 QUOTE_NONNUMERIC 格式选项（在这种情况下，未加引号的字段会转换为浮点数），否则不会执行自动数据类型转换。
with open(input_file, 'r', newline='') as filereader:#使用with，在脚本结束时，自动关闭文件对象
	with open(output_file, 'w', newline='') as filewriter:
		header = filereader.readline()#读取输入文件的第一行数据并赋值给header
		header = header.strip()#去掉两头的空格，制表符和换行符
		header_list = header.split(',')#用，拆分成列表
		print(header_list)
		filewriter.write(','.join(map(str,header_list))+'\n') #map()将str应用于header_list中的每一个元素，确保每个元素为字符串
		#join()在header_list中的每个值之间插入一个逗号','，将该list转换成一个string。
		#最后用\n换行。
		for row in filereader:#写入输出文本条件
			row = row.strip()
			row_list = row.split(',')
			print(row_list)
			filewriter.write(','.join(map(str,row_list))+'\n') #map()将str应用于row_list中的每一个元素，确保每个元素为字符串
			#join()在header_list中的每个值之间插入一个逗号','，将该list转换成一个string。
			#最后用\n换行。
```

### 使用pandas
+ to_csv()是DataFrame类的方法
+ read_csv()是pandas的方法

```
#!/usr/bin/env python3
import sys
import pandas as pd
input_file = sys.argv[1]
output_file = sys.argv[2]
data_frame = pd.read_csv(input_file) #data_frame同列表、字典、元组相似。也是储存数据的一种方式。
#data_frame保留了“表格”的数据组织方式，不需要使用列表套列表的方式来分析数据。
#即header_list = header.split(',')至filewriter.write(','.join(map(str,header_list))+'\n')
print(data_frame)
data_frame.to_csv(output_file, index=False)
```

## 读写文件part2
### 基础python 使用csv模块 
* 该模块被设计用于正确处理数据值中的嵌入逗号和其他复杂模式。
* csv.reader语法：(csvfile, dialect='excel', **fmtparams)
	* 返回一个 reader 对象，该对象将逐行遍历csvfile。
	* csvfile 可以是任何对象，只要这个对象支持 iterator 协议并在每次调用 __next__() 方法时都返回字符串，文件对象 和列表对象均适用。
	* 如果 csvfile 是文件对象，则打开它时应使用 newline=''。
	* 可选参数 dialect 是用于不同的 CSV 变种的特定参数组。它可以是 Dialect 类的子类的实例，也可以是 list_dialects() 函数返回的字符串之一。
	* 另一个可选关键字参数 fmtparams 可以覆写当前变种格式中的单个格式设置。csv 文件的每一行都读取为一个由字符串组成的列表。除非指定了 QUOTE_NONNUMERIC 格式选项（在这种情况下，未加引号的字段会转换为浮点数），否则不会执行自动数据类型转换。 
	
```
#!/usr/bin/env python3
import csv
import sys
#csv模块就是被设计用于正确处理数据值中的嵌入逗号和其他复杂模式。
input_file = sys.argv[1]
output_file = sys.argv[2]
with open(input_file, 'r', newline='') as csv_in_file:
	with open(output_file, 'w', newline='') as csv_out_file:
		###用csv 模块中的 **reader函数** 创建了一个文件读取/写入对象
		filereader = csv.reader(csv_in_file, delimiter=',')	" delimiter=',' :表','为默认分隔符。#如果输入文件是用逗号分隔则不用指定，但是此处防止输入/输出文件具有不同的分割符
		###if ur input-file and output-file both use ',' to split, u do need to use this parameter(delimiter=',')。in this case, 是为了防止所处理的input-file or output-file 有不同的分隔符，eg：';' or '\t'
		filewriter = csv.writer(csv_out_file, delimiter=',')#用csv.writer创建文件输出对象
		for row_list in filereader:
			print(row_list)
			filewriter.writerow(row_list)
```
	**delimiter：一个用于分隔字段的单字符，默认为 ','**

## 筛选特定的行
### 输入文件中筛选除特定行的3种方法：
	+ 行中的值满足某条件
	+ 行中的值属于某个集合
	+ 行中的值匹配于某个模式（正则）

- 通用结构：
```
for row_list in filereader:
    ***if value in row meets some business rule or set of rules:***
        do something
    else:
		do something else
``` 

+ 下面小节中，修改封装在***间的代码
+ csvreader.__next__()：返回 reader 的可迭代对象的下一行，返回值可能是列表（由 reader() 返回的对象）或字典（由 DictReader 返回的对象），解析是根据当前设置的变种进行的。通常应该这样调用它：next(reader)。

### 行中的值满足某个条件
#### 基础python：

```
#!/usr/bin/env python3
import csv
import sys
input_file = sys.argv[1]
output_file = sys.argv[2]
with open(input_file, 'r', newline='') as csv_in_file:
	with open(output_file, 'w', newline='') as csv_out_file:
		filereader = csv.reader(csv_in_file)
		filewriter = csv.writer(csv_out_file)
		header = next(filereader)#用next函数读取文件第一行
		filewriter.writerow(header)
		for row_list in filereader:
			supplier = str(row_list[0]).strip()#取出每行的0号位内容，去掉空格、制表符、换行符
			cost = str(row_list[3]).strip('$').replace(',', '')#去除每行的第四个值（row[3]),strip:删除$符号，将','换成' ',用str将他转换
			if supplier == 'Supplier Z' or float(cost) > 600.0:#条件
				print(row_list)
				filewriter.writerow(row_list)
```

#### pandas
 
```
#!/usr/bin/env python3
import pandas as pd
import sys
###使用了一个loc函数，同时选择特定的行和列。
###df.loc[val]：根据标签选择dataframe的单行或者多行
###df.loc[:,val]:根据标签选择单列或者多列
input_file = sys.argv[1]
output_file = sys.argv[2]
data_frame = pd.read_csv(input_file)
data_frame['Cost'] = data_frame['Cost'].str.strip("$").astype(float)
print(data_frame['Cost'])
data_frame_value_meets_condition = data_frame.loc[(data_frame['Supplier Name'].str.contains('Z')) | (data_frame['Cost'] > 600.0),:] #此处的loc[，：]应是表loc[val]##contains():表包含
print(data_frame_value_meets_condition)

```

### 行中的值属于某个集合
#### 基础python

```
#!/usr/bin/env python3
import csv
import sys
input_file = sys.argv[1]
output_file = sys.argv[2]
important_dates = ['1/20/14', '1/30/14'] #此处是列表变量，即我们所需要的集合
##创建含特定值的变量，并在后续代码中引用是很有必要的。如果变量的值发生了变化，你只需要在一处修改。
with open(input_file, 'r', newline='') as csv_in_file:
	with open(output_file, 'w', newline='') as csv_out_file:
		filereader = csv.reader(csv_in_file)
		filewriter = csv.writer(csv_out_file)
		header = next(filereader)
		filewriter.writerow(header)
		for row_list in filereader:
			a_date = row_list[4]#将每行的第五项取出赋予 a_date
			if a_date in important_dates:
				filewriter.writerow(row_list)#如果a_date在important_dates中，打印含该a_date的行

```

#### pandas

	**isin函数大法好**

```
#!/usr/bin/env python3
import pandas as pd
import sys
input_file = sys.argv[1]
output_file = sys.argv[2]
data_frame = pd.read_csv(input_file)
important_dates = ['1/20/14', '1/30/14']
data_frame_value_in_set = data_frame.loc[data_frame['Purchase Date'].isin(important_dates), :]#isin函数大法好
data_frame_value_in_set.to_csv(output_file, index=False)
```


### 行中的值匹配与某个模式（正则）
#### 基础python
+ 正则有几个小可爱
	 + re.compile函数：将文本形式的模式编译成编译后的正则表达式
	 + re.I函数：确保模式不区分大小写
	 + r : 确保python不处理字符串之间的转义符eg：\t、\、\n 

```#!/usr/bin/env python3
import csv
import re #如果想用正则请import re
import sys
input_file = sys.argv[1]
output_file = sys.argv[2]
pattern = re.compile(r'(?P<my_pattern_group>^001-.*)', re.I)
##### re.compile:创建一个名为pattern的正则表达式变量
##### r: 将''之间的模式当作原始字符，即不处理'(?P<my_pattern_group>^001-.*)'字符串之间的转义符
##### ?P<my_pattern_group>：元字符。捕获了名为<my_pattern_group>的组中匹配了的子字符串
##### ^001-.*
##### ^ : 以001-开头，
##### . : 表可以匹配任何字符，除换行符\n
##### * : 表重复前的字符>=0次
##### .* : 除换行符之外的任意字符可以在"001-"之后出现任意次 or 只要字符开始是001-匹配正则表达式
with open(input_file, 'r', newline='') as csv_in_file:
	with open(output_file, 'w', newline='') as csv_out_file:
		filereader = csv.reader(csv_in_file)
		filewriter = csv.writer(csv_out_file)
		header = next(filereader)
		filewriter.writerow(header)
		for row_list in filereader:
			invoice_number = row_list[1]
			if pattern.search(invoice_number):
				print(row_list)
				filewriter.writerow(row_list)
```

#### pandas
	+ 用starwith函数代替正则
t
```
#!/usr/bin/env python3
import pandas as pd
import sys
input_file = sys.argv[1]
output_file = sys.argv[2]
data_frame = pd.read_csv(input_file)
data_frame_value_matches_pattern = data_frame.loc[data_frame['Invoice Number'].str.startswith("001-"), :]#我跪了，pandas真香
data_frame_value_matches_pattern.to_csv(output_file, index=False)
```

## 选取特定的列
- 两种方法
	- 使用列索引值
	 - 使用列标题
### 列索引值
#### 基础python

```
#!/usr/bin/env python3
import csv
import sys
input_file = sys.argv[1]
output_file = sys.argv[2]
my_columns = [0, 3]#定义变量，给索引值index_value创造变量条件
#此处表示0号位和3号位，不是0~3。如果是0~3应该是range（0，4）
with open(input_file, 'r', newline='') as csv_in_file:
	with open(output_file, 'w', newline='') as csv_out_file:
		filereader = csv.reader(csv_in_file)
		filewriter = csv.writer(csv_out_file)
		for row_list in filereader:
			row_list_output = [ ]
			for index_value in my_columns:
				row_list_output.append(row_list[index_value])
			filewriter.writerow(row_list_output)
			print(row_list_output)
```

#### pandas
+ 使用iloc()函数，来根据位置选取索引
	 + df.iloc[where]:根据整数位置选择单行或者多行
	 + df.iloc[:,where]:根据整数位置选择单列或者多列
	 + df.iloc[where_i,where_j]:根据整数位置选择行或列
```
#!/usr/bin/env python3
import pandas as pd
import sys
input_file = sys.argv[1]
output_file = sys.argv[2]
data_frame = pd.read_csv(input_file)
data_frame_column_by_index = data_frame.iloc[:, [0, 3]]#选择位置为0和3的两列
data_frame_column_by_index.to_csv(output_file, index=False)#index=False:输出不显示index(索引)值
```
### 列标题
#### 基础python

```
#!/usr/bin/env python3
import csv
import sys
input_file = sys.argv[1]
output_file = sys.argv[2]
my_columns = ['Invoice Number', 'Purchase Date']
my_columns_index = [ ]
with open(input_file, 'r', newline='') as csv_in_file:
	with open(output_file, 'w', newline='') as csv_out_file:
		filereader = csv.reader(csv_in_file)
		filewriter = csv.writer(csv_out_file)
		header = next(filereader,None)#加减None对此例输出无影响，但是网络搜索，None是为了排错
		####原话如下：“next是取下一个元素嘛，如果你一直取到最后一个，又不小心取了下一个，不加none会报错，加了none那下一个不存在的元素就是none”
		for index_value in range(len(header)):
			if header[index_value] in my_columns:
				my_columns_index.append(index_value)
		filewriter.writerow(my_columns)#文件加入了标题
		for row_list in filereader:
			row_list_output = [ ]#建一个空表
			for index_value in my_columns_index:
				row_list_output.append(row_list[index_value])#将数据填进去
			filewriter.writerow(row_list_output)#将数据加入文件
			print(row_list_output)#打印出的只是数据
```

#### pandas

```
#!/usr/bin/env python3
import pandas as pd
import sys
#loc函数好
#pandas真香
#df.loc[val]：根据标签选择dataframe的单行或者多行
#df.loc[:,val]:根据标签选择单列或者多列
input_file = sys.argv[1]
output_file = sys.argv[2]
data_frame = pd.read_csv(input_file)
data_frame_column_by_name = data_frame.loc[:, ['Invoice Number', 'Purchase Date']]#df.loc[:,val]:根据标签选择单列或者多列
data_frame_column_by_name.to_csv(output_file, index=False)
```

## 选取连续的行
### 基础python

```
#!/usr/bin/env python3
import csv
import sys
input_file = sys.argv[1]
output_file = sys.argv[2]
row_counter = 0
with open(input_file, 'r', newline='') as csv_in_file:
	with open(output_file, 'w', newline='') as csv_out_file:
		filereader = csv.reader(csv_in_file)
		filewriter = csv.writer(csv_out_file)
		for row in filereader:
			if row_counter >= 3 and row_counter <= 15:#要选取4~15行，因为有0号位，所以范围是3~15###可用range(3,16),range 它不香么
			
				filewriter.writerow([value.strip() for value in row])#顺手处理下value让它更可爱,然后将处理好的value填入filewriter
				print(row_counter, [value.strip() for value in row])#该行的缩进应该和filewriter.writerow一致，不然会缺少if的判断
			row_counter += 1
```

### pandas

```
#!/usr/bin/env python3
import pandas as pd
import sys
#drop函数
input_file = sys.argv[1]
output_file = sys.argv[2]
data_frame = pd.read_csv(input_file, header=None)#header=None:告诉函数，我们读取的原始文件数据没有列索引。因此，read_csv为自动加上列索引。
data_frame = data_frame.drop([0,1,2,16,17,18])#drop [0,1,2,16,17,18]这几列，此时header还存在自动默认的列索引
data_frame.columns = data_frame.iloc[0]#根据第0行索引，作为列索引。#就是将默认生成的列索引改为data_frame的第0行
data_frame = data_frame.reindex(data_frame.index.drop(3))#用reindex重新为数据框生成索引#如果不drop(3)，输出会多一行index为3的一行。
###此处删掉的是data_frame的第0行，留下的是默认生成的被修改过后的列索引
data_frame.to_csv(output_file, index=False)
```

## 添加标题行
### 基础python

```
#!/usr/bin/env python3
import csv
import sys
input_file = sys.argv[1]
output_file = sys.argv[2]
with open(input_file, 'r', newline='') as csv_in_file:
	with open(output_file, 'w', newline='') as csv_out_file:
		filereader = csv.reader(csv_in_file)
		filewriter = csv.writer(csv_out_file)
		header_list = ['Supplier Name', 'Invoice Number','Part Number', 'Cost', 'Purchase Date']#徒手插title
		filewriter.writerow(header_list)
		for row in filereader:
			filewriter.writerow (row)
```

### pandas

```
#!/usr/bin/env python3
import pandas as pd
import sys
input_file = sys.argv[1]
output_file = sys.argv[2]
header_list = ['Supplier Name', 'Invoice Number','Part Number', 'Cost', 'Purchase Date']
data_frame = pd.read_csv(input_file, header=None, names=header_list)
#names : array-like, default None
#names用于结果的列名列表，如果数据文件中没有列标题行，就需要执行header=None。默认列表中不能出现重复，除非设定参数mangle_dupe_cols=True。
data_frame.to_csv(output_file, index=False)
print(data_frame)
```

## 读取多个csv文件
	+ 唔 这儿整了一个glob。。。
+ 此处演示了如何读取多个csv文件并将文件的**基本信息**打印到屏幕上。

### 基本python

```
#!/usr/bin/env python3
import csv
import glob #该模块可以定位匹配与某个特定模式的所有路径名
import os #该模块包含了用于解析路径名的函数。
import sys
input_path = sys.argv[1]
file_counter = 0
for input_file in glob.glob(os.path.join(input_path,'sales_*')):# *这个通配符，sales_*：表以sales_开头的,_后面可以是任意字符的文件。
###想找全部某个类型的文件，eg：csv。用 *.csv.
###os.path.join()函数：将（）内的两个连接在一起【input_path是路径，'sales_*'表以sales_开头的,_后面可以是任意字符的文件】
### 即以下文件：C:\Users\86152\Desktop\csv\sales_january_2014.csv , C:\Users\86152\Desktop\csv\sales_february_2014.csv , C:\Users\86152\Desktop\csv\sales_march_2014.csv
	row_counter = 1
	with open(input_file, 'r', newline='') as csv_in_file:
		filereader = csv.reader(csv_in_file)
		header = next(filereader)
		for row in filereader:
			row_counter += 1
	print('{0!s}: \t{1:d} rows \t{2:d} columns'.format(os.path.basename(input_file), row_counter, len(header)))
	file_counter += 1
print('Number of files: {0:d}'.format(file_counter))
###{0!s}:输入文件名
###{1:d} rows：输入文件行数
###{2:d} columns：输入文件列数
###os.path.basename(input_file)函数：表从完整路径名中抽取基本文件名
###inputfile是path即路径，所以sys.argv[1]处输入的是路径
```

## 从多个文件中链接数据
### 基础python
+ 在'a'下，不要重复执行terminal命令，输出文件会不断叠加内容

```
#!/usr/bin/env python3
import csv
import glob
import os
import sys
input_path = sys.argv[1]
output_file = sys.argv[2]
first_file = True
for input_file in glob.glob(os.path.join(input_path,'sales_*')):
	print(os.path.basename(input_file))
	with open(input_file, 'r', newline='') as csv_in_file:
		with open(output_file, 'a', newline='') as csv_out_file:#a是追加哦~~看一眼啊~如果是w就会覆盖哦~想好需求呢
			filereader = csv.reader(csv_in_file)
			filewriter = csv.writer(csv_out_file)
			if first_file:#该判断用来区分是否是第一份文件。目的是让标题仅写入一遍。
				for row in filereader:
					filewriter.writerow(row)
				first_file = False
			else:#将余下的所有文件输入
				header = next(filereader)#此处将所有文件的标题赋予给变量header。让后续的for循环处理跳过标题行。
				for row in filereader:
					filewriter.writerow(row)
```

+ 输出的csv文件集合了所有输入文件的行

### pandas
+ concat函数大法好！concat函数会将所有数据框连接成一个数据框（所以可能是不要做header的原因）
	+ concat使用axis参数来设置连接数据框的方式
		+ axis = 0 ：表从头到尾垂直堆叠（竖着叠）
		+ axis = 1 : 表并排地平行堆叠（横着加）

```
#!/usr/bin/env python3
import pandas as pd
import glob
import os
import sys
input_path = sys.argv[1]
output_file = sys.argv[2]
all_files = glob.glob(os.path.join(input_path,'sales_*'))
all_data_frames = [ ]
for file in all_files:
	data_frame = pd.read_csv(file, index_col=None)
	all_data_frames.append(data_frame)
data_frame_concat = pd.concat(all_data_frames, axis=0, ignore_index=True) # axis = 0 ：表从头到尾垂直堆叠
###ignore_index=True 对index重新安排, 为False的时候会保留之前的index
data_frame_concat.to_csv(output_file, index = False)##index = False：输出不显示index(索引)值
```

###  这是一个小插曲
**Merge多个DataFrame**
```
res= pd.merge(df1, df2, on='key')
pd.merge(df1, df2, on=['key1', 'key2'])
```

+ 可以指定合并的方法，
	+ inner，key的值必须一样的才和并
	+ outer, 不管值是不是一样都会合并， 对于不一样的值填充NAN
	+ left&right 只靠一边的数据，另一边不一样的填充NAN
+ indicator, 显示是如何merge的
+ index, 通过对比index来进行merge

## 计算每个文件中值的总和与平均值

### 基础python

```
#!/usr/bin/env python3
import csv
import glob
import os
import string
import sys
input_path = sys.argv[1]
output_file = sys.argv[2]
output_header_list = ['file_name', 'total_sales', 'average_sales']#创建了一个输出文件的列标题列表
csv_out_file = open(output_file, 'a', newline='')
filewriter = csv.writer(csv_out_file)
filewriter.writerow(output_header_list)
for input_file in glob.glob(os.path.join(input_path,'sales_*')):
	with open(input_file, 'r', newline='') as csv_in_file:
		filereader = csv.reader(csv_in_file)
		output_list = [ ]
		output_list.append(os.path.basename(input_file))
		header = next(filereader)#除去每个输入文件的标题行
		total_sales = 0.0
		number_of_sales = 0.0
		for row in filereader:#读取所有文件的行
			sale_amount = row[3]
			total_sales += float(str(sale_amount).strip('$').replace(',',''))#此时开始数据处理
			##已经获取了了每行第3号位的值，转str，去$,将','去掉。最后将其转化为float
			number_of_sales += 1.0 #注意：因为上述是float，整个脚本的数字最好格式相同
		average_sales = '{0:.2f}'.format(total_sales / number_of_sales) #简单的计算过程，并且将结果保留两位小数
		output_list.append(total_sales)
		output_list.append(average_sales)
		filewriter.writerow(output_list)
		print(output_list)
csv_out_file.close()
```

### pandas
+ 讲真的，有sum和mean函数，为什么要为难自己写python

```
#!/usr/bin/env python3
import pandas as pd
import glob
import os
import sys
input_path = sys.argv[1]
output_file = sys.argv[2]
all_files = glob.glob(os.path.join(input_path,'sales_*'))
all_data_frames = []
for input_file in all_files:
	data_frame = pd.read_csv(input_file, index_col=None)
	total_sales = pd.DataFrame([float(str(value).strip('$').replace(',','')) for value in data_frame.loc[:, 'Sale Amount']]).sum()#还是要处理一下数据，让函数能用
	###loc：取了Sale Amount的那一列
	average_sales = pd.DataFrame([float(str(value).strip('$').replace(',','')) for value in data_frame.loc[:, 'Sale Amount']]).mean()
	data = {'file_name': os.path.basename(input_file),'total_sales': total_sales,'average_sales': average_sales}
	all_data_frames.append(pd.DataFrame(data, columns=['file_name', 'total_sales', 'average_sales']))
data_frames_concat = pd.concat(all_data_frames, axis=0, ignore_index=True)#axis=0 竖着堆，ignore_index=True
###ignore_index=True 对index重新安排, 为False的时候会保留之前的index
data_frames_concat.to_csv(output_file, index = False)##index = False:输出不显示index(索引)值
print(data_frames_concat)
```
















		

	
	
	












