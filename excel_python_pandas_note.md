# excel 文件
- 思考如何将pandas中输出文件的time去掉
##  处理excel 要先import扩展包
	+ python没有处理excel的标准模块，所以当使用python处理excel时，要安装xlrd和xlwt两个扩展包pip install一下
	+ 完整版（pip install 包名 -i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com）
		+ from xlwt import * 
		+ from xlrd import *
	+ xlrd意为:xls文件read库，只能读。
	+ 若写入，要用xlwt，意为:xls文件write写入库。
## 内省excel工作簿
	+ 首先excel文本不是纯文本文件，所以不能在文本编辑器中打开并查看数据
	+ excel工作簿 = excel文件：里面包含一个或者多个excel表
	
### 确定工作簿中 工作表的数量、名称以及各个工作表中行列的数量。
```
#!/usr/bin/env python3
import sys
from xlrd import *    # 为了避免麻烦，这边之间用*代替所有函数，该节代码时为了展示 open_workbook函数
input_file = sys.argv[1]
workbook = open_workbook(input_file)
print('Number of worksheets:', workbook.nsheets)#nsheets：number of the sheets
for worksheet in workbook.sheets():
	print("Worksheet name:", worksheet.name, "\tRows:", worksheet.nrows, "\tColumns:", worksheet.ncols)

```

# 处理单个工作表
## 读写excel文件
### 基础python和xlrd 、xlwt模块

```
#!/usr/bin/env python3
###此处开始不改import，用来明确该节代码主要是讲的什么函数
import sys
from xlrd import open_workbook
from xlwt import Workbook
input_file = sys.argv[1]
output_file = sys.argv[2]

output_workbook = Workbook() # 实例化一个 xlwt Workbook对象，来让结果写入用于输出的excel文件
output_worksheet = output_workbook.add_sheet('jan_2013_output') ##用add_sheet（）函数 为输出的工作簿添加一个工作表名为jan_2013_output

with open_workbook(input_file) as workbook:##用xlrd 的 open_workbook函数，打开用于输入的工作簿，并将结果赋予一个workbook对象。
	worksheet = workbook.sheet_by_name('january_2013')## 用workbook对象的 sheet_by_name（）函数：表引用名为january_2013的工作表。
	for row_index in range(worksheet.nrows):
		for column_index in range(worksheet.ncols):
			output_worksheet.write(row_index, column_index, worksheet.cell_value(row_index, column_index))##使用xlwt的write和行列的索引将每个单元格的值写入输出文件的工作表
output_workbook.save(output_file)

```

### 上述这段代码日期输出为数值，不是日期。
+ excel 将日期和时间保存为浮点数，该浮点数表示从1900年1月0日开始经过的日期数，加上24小时的小数部分。
	+ eg：1 表示 1900年1月1日，因为过去了一天，
+ xlrd 提供了函数来格式化日期值。下段代码展示

### 格式化日期数据
```
#!/usr/bin/env python3
import sys
from datetime import date #datetime模块专门处理时间
from xlrd import open_workbook, xldate_as_tuple#xldate_as_tuple函数：可以将excel中代表日期、时间或日期时间的数值转换为元组。
###只要将数值转化成元组，便可以提取出具体时间元素并格式化成不同时间格式
from xlwt import Workbook
input_file = sys.argv[1]
output_file = sys.argv[2]
output_workbook = Workbook()#实例化一个 xlwt Workbook对象，来让结果写入用于输出的excel文件
output_worksheet = output_workbook.add_sheet('jan_2013_output') ##用add_sheet（）函数 为输出的工作簿添加一个工作表名为jan_2013_output
with open_workbook(input_file) as workbook:##用xlrd 的 open_workbook函数，打开用于输入的工作簿，并将结果赋予一个workbook对象。
	worksheet = workbook.sheet_by_name('january_2013')## 用workbook对象的 sheet_by_name（）函数：表引用名为january_2013的工作表。
	for row_index in range(worksheet.nrows):
		row_list_output = []
		for col_index in range(worksheet.ncols):
			if worksheet.cell_type(row_index, col_index) == 3:#检查在(row_index, col_index)这个单元格类型是否为数字3，？？3 为什么表示日期数据？？!!!先记着==3表日期好了
				###该if-else检验每个单元格是否含有日期数据
				date_cell = xldate_as_tuple(worksheet.cell_value(row_index, col_index),workbook.datemode)#cell_value == cell().value
				###这个单元格的值作为xldate_as_tuple函数中的第一个参数，会被转换成元组中的一个代表日期的浮点数
				###workbook.datemode是**必须的**，它可以使函数确定日期是基于1900还是1904（mac上有的版本是基于1904的）
				###workbook.datemode函数的结果被赋给一个元组变量date_cell
				date_cell = date(*date_cell[0:3]).strftime('%m/%d/%Y')##strftime函数将date对象转换为一个具有特定格式('%m/%d/%Y')的字符串
				print(date_cell)
				row_list_output.append(date_cell)
				output_worksheet.write(row_index, col_index, date_cell)
			else:###else代码处处理所有的非日期块
				non_date_cell = worksheet.cell_value(row_index,col_index)
				row_list_output.append(non_date_cell)
				output_worksheet.write(row_index, col_index,non_date_cell)
output_workbook.save(output_file)
```
**上节代码感觉是在洗date这个数据，将date改成所需要的类型**

### pandas
#### 读取excel文件的函数
```
#!/usr/bin/env python3
import pandas as pd
import sys
input_file = sys.argv[1]
output_file = sys.argv[2]
data_frame = pd.read_excel(input_file, sheet_name='january_2013')
writer = pd.ExcelWriter(output_file)
data_frame.to_excel(writer, sheet_name='jan_13_output', index=False)
writer.save()
```
**上段代码只是读取excel的数据，日期部分显示出日期+时间（00：00：00）**

## 筛选特定行
### 行中满足某个条件
#### 基础python

```
#!/usr/bin/env python3
import sys
from datetime import date
from xlrd import open_workbook, xldate_as_tuple
from xlwt import Workbook
input_file = sys.argv[1]
output_file = sys.argv[2]
output_workbook = Workbook()#实例化一个 xlwt Workbook对象，来让结果写入用于输出的excel文件
output_worksheet = output_workbook.add_sheet('jan_2013_output')##用add_sheet（）函数 为输出的工作簿添加一个工作表名为jan_2013_output
sale_amount_column_index = 3 #用来定位sale amount列
with open_workbook(input_file) as workbook:##用xlrd 的 open_workbook函数，打开用于输入的工作簿，并将结果赋予一个workbook对象。
	worksheet = workbook.sheet_by_name('january_2013')## 用workbook对象的 sheet_by_name（）函数：表引用名为january_2013的工作表。
	data = []
	header = worksheet.row_values(0)#提取出标题行中的值
	data.append(header)
	for row_index in range(1,worksheet.nrows):
			row_list = []
			sale_amount = worksheet.cell_value(row_index, sale_amount_column_index)
			if sale_amount > 1400.0:
				for column_index in range(worksheet.ncols):
					cell_value = worksheet.cell_value(row_index,column_index)
					cell_type = worksheet.cell_type(row_index, column_index)
					if cell_type == 3:#检验是否为日期类型
						date_cell = xldate_as_tuple(cell_value,workbook.datemode)
						###这个单元格的值作为xldate_as_tuple函数中的第一个参数，会被转换成元组中的一个代表日期的浮点数
						###workbook.datemode是**必须的**，它可以使函数确定日期是基于1900还是1904（mac上有的版本是基于1904的）
						###workbook.datemode函数的结果被赋给一个元组变量date_cell
						date_cell = date(*date_cell[0:3]).strftime('%m/%d/%Y')##strftime函数将date对象转换为一个具有特定格式('%m/%d/%Y')的字符串
						row_list.append(date_cell)
					else:
						row_list.append(cell_value)
			if row_list:#检验row_list是否为空
				data.append(row_list)##只将非空row_list添加进data

	for list_index, output_list in enumerate(data):
		for element_index, element in enumerate(output_list):###enumerate() 函数:用于将一个可遍历的数据对象(如列表、元组或字符串)组合为一个索引序列，同时列出数据和数据下标，一般用在 for 循环当中。
			output_worksheet.write(list_index, element_index, element)

output_workbook.save(output_file)
```
#### pandas
**如果你需要设定多个条件，可以将条件放于圆括号（）中，用& 或者 | 连接起来**
+ & 表两个都为正即 and
+ | 表一个条件为真即 or
```
#!/usr/bin/env python3
import pandas as pd
import sys
input_file = sys.argv[1]
output_file = sys.argv[2]
data_frame = pd.read_excel(input_file, 'january_2013', index_col=None)##index_col=None：表重新设置一列成为index值
###index_col=False：重新设置一列成为index值
###index_col=0：第一列为index值
###pd.read_excel(io（excel路径）, sheetname=0,header=0,skiprows=None,index_col=None,
				###names=None,arse_cols=None,date_parser=None,na_values=None,thousands=None,
				###convert_float=True,has_index_names=None,converters=None,dtype=None,
data_frame_value_meets_condition = data_frame[data_frame['Sale Amount'].astype(float) > 1400.0]
writer = pd.ExcelWriter(output_file)
data_frame_value_meets_condition.to_excel(writer, sheet_name='jan_13_output', index=False)
###to_excel(self, excel_writer, sheet_name='Sheet1', na_rep='', float_format=None,columns=None,merge_cells=True, encoding=None,inf_rep='inf', verbose=True, freeze_panes=None)
writer.save()

```

##### **还是会将time输出**

### 行中的值属于某个集合
 
#### 基础python
##### **和前面的csv有点儿像**

```
#!/usr/bin/env python3
import sys
from datetime import date
from xlrd import open_workbook, xldate_as_tuple
from xlwt import Workbook
input_file = sys.argv[1]
output_file = sys.argv[2]
output_workbook = Workbook()#实例化一个 xlwt Workbook对象，来让结果写入用于输出的excel文件
output_worksheet = output_workbook.add_sheet('jan_2013_output')##用add_sheet（）函数 为输出的工作簿添加一个工作表名为jan_2013_output
important_dates = ['01/24/2013', '01/31/2013']#同csv变量集合的概念
purchase_date_column_index = 4 #用来定位purchase_date列
with open_workbook(input_file) as workbook:##用xlrd 的 open_workbook函数，打开用于输入的工作簿，并将结果赋予一个workbook对象。
	worksheet = workbook.sheet_by_name('january_2013')## 用workbook对象的 sheet_by_name（）函数：表引用名为january_2013的工作表。
	data = []
	header = worksheet.row_values(0)
	data.append(header)
	for row_index in range(1, worksheet.nrows):		
		purchase_datetime = xldate_as_tuple(worksheet.cell_value(row_index, purchase_date_column_index),workbook.datemode)###这个单元格的值作为xldate_as_tuple函数中的第一个参数，会被转换成元组中的一个代表日期的浮点数
						###workbook.datemode是**必须的**，它可以使函数确定日期是基于1900还是1904（mac上有的版本是基于1904的）
						###workbook.datemode函数的结果被赋给一个元组变量date_cell
		purchase_date = date(*purchase_datetime[0:3]).strftime('%m/%d/%Y')#用来匹配important_dates中格式化的日期
		row_list = []
		if purchase_date in important_dates:##检验是否是important_dates的日期
			for column_index in range(worksheet.ncols):
				cell_value = worksheet.cell_value(row_index,column_index)
				cell_type = worksheet.cell_type(row_index, column_index)
				if cell_type == 3:
					date_cell = xldate_as_tuple(cell_value,workbook.datemode)
					date_cell = date(*date_cell[0:3]).strftime('%m/%d/%Y')##strftime函数将date对象转换为一个具有特定格式('%m/%d/%Y')的字符串
					row_list.append(date_cell)
				else:
					row_list.append(cell_value)
		if row_list:
			data.append(row_list)

	for list_index, output_list in enumerate(data):###enumerate() 函数:用于将一个可遍历的数据对象(如列表、元组或字符串)组合为一个索引序列，同时列出数据和数据下标，一般用在 for 循环当中。
		for element_index, element in enumerate(output_list):
			output_worksheet.write(list_index, element_index, element)
output_workbook.save(output_file)

```

#### pandas
+ 让我们 再一次 isin 函数大法好！
```
#!/usr/bin/env python3
import pandas as pd
import string
import sys

input_file = sys.argv[1]
output_file = sys.argv[2]
data_frame = pd.read_excel(input_file, 'january_2013', index_col=None)
important_dates = ['01/24/2013','01/31/2013']
data_frame_value_in_set = data_frame[data_frame['Purchase Date'].isin(important_dates)]
writer = pd.ExcelWriter(output_file)
data_frame_value_in_set.to_excel(writer, sheet_name='jan_13_output', index=False)
writer.save()
```
###  行中的值匹配于特定模式
#### 基础python

```
#!/usr/bin/env python3
import re##去你的特定模式，还不是逃不过正则
import sys
from datetime import date
from xlrd import open_workbook, xldate_as_tuple
from xlwt import Workbook
input_file = sys.argv[1]
output_file = sys.argv[2]
output_workbook = Workbook()
output_worksheet = output_workbook.add_sheet('jan_2013_output')
pattern = re.compile(r'(?P<my_pattern>^J.*)')
##### re.compile:创建一个名为pattern的正则表达式变量
##### r: 将''之间的模式当作原始字符，即不处理'(?P<my_pattern_group>^001-.*)'字符串之间的转义符
##### ?P<my_pattern>：元字符。捕获了名为<my_pattern>的组中匹配了的子字符串
##### ^J.*
##### ^ : 以J开头，
##### . : 表可以匹配任何字符，除换行符\n
##### * : 表重复前的字符>=0次
##### .* : 除换行符之外的任意字符可以在"J"之后出现任意次 or 只要字符开始是J匹配正则表达式
customer_name_column_index = 1
with open_workbook(input_file) as workbook:
	worksheet = workbook.sheet_by_name('january_2013')
	data = []
	header = worksheet.row_values(0)
	data.append(header)
	for row_index in range(1, worksheet.nrows):		
		row_list = []
		if pattern.search(worksheet.cell_value(row_index, customer_name_column_index)):#re模块的search函数在customer name列中搜索模式，并检测是否能找到一个匹配的
			###如果找到一个匹配的，就将这一行中的每一个值添加进row_list中
			for column_index in range(worksheet.ncols):
				cell_value = worksheet.cell_value(row_index,column_index)
				cell_type = worksheet.cell_type(row_index, column_index)
				if cell_type == 3:
					date_cell = xldate_as_tuple(cell_value,workbook.datemode)###这个单元格的值作为xldate_as_tuple函数中的第一个参数，会被转换成元组中的一个代表日期的浮点数
						###workbook.datemode是**必须的**，它可以使函数确定日期是基于1900还是1904（mac上有的版本是基于1904的）
						###workbook.datemode函数的结果被赋给一个元组变量date_cell
					date_cell = date(*date_cell[0:3]).strftime('%m/%d/%Y')##strftime函数将date对象转换为一个具有特定格式('%m/%d/%Y')的字符串
					row_list.append(date_cell)
				else:
					row_list.append(cell_value)
		if row_list:
			data.append(row_list)
	for list_index, output_list in enumerate(data):###enumerate() 函数:用于将一个可遍历的数据对象(如列表、元组或字符串)组合为一个索引序列，同时列出数据和数据下标，一般用在 for 循环当中。
		for element_index, element in enumerate(output_list):
			output_worksheet.write(list_index, element_index, element)
output_workbook.save(output_file)
```

#### pandas
+ 就问问你**startwith、endswith、match、search**这些函数他不香么
```
#!/usr/bin/env python3
import pandas as pd
import sys
input_file = sys.argv[1]
output_file = sys.argv[2]
data_frame = pd.read_excel(input_file, 'january_2013', index_col=None)
data_frame_value_matches_pattern = data_frame[data_frame['Customer Name'].str.startswith("J")]
writer = pd.ExcelWriter(output_file)
data_frame_value_matches_pattern.to_excel(writer, sheet_name='jan_13_output', index=False)
writer.save()
```
## 选取特定列
#### 两种方法
- 使用列索引值
	- 使用列标题

#### 使用列索引值

#### 基础python
+ emmm没啥好说的，重复的内容，但是还是要理解哦~~~
```
#!/usr/bin/env python3
import sys
from datetime import date
from xlrd import open_workbook, xldate_as_tuple
from xlwt import Workbook
input_file = sys.argv[1]
output_file = sys.argv[2]
output_workbook = Workbook()
output_worksheet = output_workbook.add_sheet('jan_2013_output')
my_columns = [1, 4]#设置变量，即所想要的列的索引值
with open_workbook(input_file) as workbook:##用xlrd 的 open_workbook函数，打开用于输入的工作簿，并将结果赋予一个workbook对象。
	worksheet = workbook.sheet_by_name('january_2013')## 用workbook对象的 sheet_by_name（）函数：表引用名为january_2013的工作表。
	data = []
	for row_index in range(worksheet.nrows):
		row_list = []
		for column_index in my_columns:
			cell_value = worksheet.cell_value(row_index,column_index)
			cell_type = worksheet.cell_type(row_index, column_index)
			if cell_type == 3:
				date_cell = xldate_as_tuple(cell_value,workbook.datemode)
				date_cell = date(*date_cell[0:3]).strftime('%m/%d/%Y')
				row_list.append(date_cell)
			else:
				row_list.append(cell_value)
		data.append(row_list)
	for list_index, output_list in enumerate(data):
		for element_index, element in enumerate(output_list):
			output_worksheet.write(list_index, element_index, element)
output_workbook.save(output_file)
```

#### pandas
+ pandas这个小可爱有很多中选取特定列的办法，比如设置数据框，在方括号中保留的列的索引值或名称(字符串)。
+ 另一种就是下面这个代码了，即 设置数据库和iloc函数。
+ 此时我们回顾一下iloc这个小可爱：
   + 使用iloc()函数，来根据位置选取索引
	 + df.iloc[where]:根据整数位置选择单行或者多行
	 + df.iloc[:,where]:根据整数位置选择单列或者多列
	 + df.iloc[where_i,where_j]:根据整数位置选择行或列
```
#!/usr/bin/env python3
import pandas as pd
import sys
###df.iloc[:,where]:根据整数位置选择单列或者多列
input_file = sys.argv[1]
output_file = sys.argv[2]
data_frame = pd.read_excel(input_file, 'january_2013', index_col=None)
data_frame_column_by_index = data_frame.iloc[:, [1, 4]]##你看这个iloc它又大又圆，是不是索引1号和4号列。
writer = pd.ExcelWriter(output_file)
data_frame_column_by_index.to_excel(writer, sheet_name='jan_13_output', index=False)
writer.save()
```
#### 列标题
#### 基础python

```
#!/usr/bin/env python3
import sys
from datetime import date
from xlrd import open_workbook, xldate_as_tuple
from xlwt import Workbook
input_file = sys.argv[1]
output_file = sys.argv[2]
output_workbook = Workbook()
output_worksheet = output_workbook.add_sheet('jan_2013_output')
my_columns = ['Customer ID', 'Purchase Date']#你看，这就是要动的列的列标题
with open_workbook(input_file) as workbook:
	worksheet = workbook.sheet_by_name('january_2013')
	data = [my_columns]
	header_list = worksheet.row_values(0)#提取出标题行中的值
	header_index_list = []
	for header_index in range(len(header_list)):
		if header_list[header_index] in my_columns:#使用列表索引来检验每个列标题是否在my_columns中。
			header_index_list.append(header_index)##如果是，则写入header_index_list
	for row_index in range(1,worksheet.nrows):
		row_list = []
		for column_index in header_index_list:
			cell_value = worksheet.cell_value(row_index,column_index)
			cell_type = worksheet.cell_type(row_index, column_index)
			if cell_type == 3:
				date_cell = xldate_as_tuple(cell_value,workbook.datemode)
				date_cell = date(*date_cell[0:3]).strftime('%m/%d/%Y')
				row_list.append(date_cell)
			else:
				row_list.append(cell_value)
		data.append(row_list)
	for list_index, output_list in enumerate(data):
		for element_index, element in enumerate(output_list):
			output_worksheet.write(list_index, element_index, element)
output_workbook.save(output_file)
```
#### pandas
+ loc你好，此处要使用loc函数的话，需要在列标题列表前面加上一个冒号和一个逗号，表你想为这些特定的列保留所有行
+ 来，我们再来回顾一下可爱的loc函数
	+ 使用了一个loc函数，同时选择特定的行和列。
	+ df.loc[val]：根据标签选择dataframe的单行或者多行
	+ df.loc[:,val]:根据标签选择单列或者多列
```
#!/usr/bin/env python3
import pandas as pd
import sys
input_file = sys.argv[1]
output_file = sys.argv[2]
data_frame = pd.read_excel(input_file, 'january_2013', index_col=None)
data_frame_column_by_name = data_frame.loc[:, ['Customer ID', 'Purchase Date']]#在列标题列表前面加上一个冒号和一个逗号，表你想为这些特定的列保留所有行
writer = pd.ExcelWriter(output_file)
data_frame_column_by_name.to_excel(writer, sheet_name='jan_13_output', index=False)
writer.save()
```
## 读取工作簿中所有的工作表
### 在所有工作表中删选特定行
#### 基础python
##### 感觉也没啥，就是迭代了一下worksheet，等于将所有的工作表内容读出来，然后再去掉标题行，处理数据行
```
#!/usr/bin/env python3
import sys
from datetime import date
from xlrd import open_workbook, xldate_as_tuple
from xlwt import Workbook
input_file = sys.argv[1]
output_file = sys.argv[2]
output_workbook = Workbook()
output_worksheet = output_workbook.add_sheet('filtered_rows_all_worksheets')
sales_column_index = 3 #保存sale amount列的索引值
threshold = 2000.0
first_worksheet = True
with open_workbook(input_file) as workbook:##用xlrd 的 open_workbook函数，打开用于输入的工作簿，并将结果赋予一个workbook对象。
	data = []
	for worksheet in workbook.sheets():#工作簿之间所有工作表的迭代
		if first_worksheet:
			header_row = worksheet.row_values(0)
			data.append(header_row)
			first_worksheet = False
		for row_index in range(1,worksheet.nrows):
			row_list = []
			sale_amount = worksheet.cell_value(row_index, sales_column_index)
			sale_amount = float(str(sale_amount).replace('$', '').replace(',', ''))
			if sale_amount > threshold:
				for column_index in range(worksheet.ncols):
					cell_value = worksheet.cell_value(row_index,column_index)
					cell_type = worksheet.cell_type(row_index, column_index)
					if cell_type == 3:
						date_cell = xldate_as_tuple(cell_value,workbook.datemode)
						date_cell = date(*date_cell[0:3]).strftime('%m/%d/%Y')
						row_list.append(date_cell)
					else:
						row_list.append(cell_value)
			if row_list:
				data.append(row_list)
	for list_index, output_list in enumerate(data):
		for element_index, element in enumerate(output_list):
			output_worksheet.write(list_index, element_index, element)
output_workbook.save(output_file)

```
#### pandas
##### 通过read_excel函数中设置sheetname=None可以读取工作簿中的所有工作表。
```
#!/usr/bin/env python3
import pandas as pd
import sys

input_file = sys.argv[1]
output_file = sys.argv[2]
data_frame = pd.read_excel(input_file, sheet_name=None, index_col=None)##读取工作簿中的所有工作表。
row_output = []
for worksheet_name, data in data_frame.items():##字典key&value的读取
	data['Sale Amount'] = data['Sale Amount'].replace(r'$', '')
	data['Sale Amount'] = data['Sale Amount'].replace(r',', '')
	data['Sale Amount'] = data['Sale Amount'].astype(float)
	row_output.append(data[data['Sale Amount'].astype(float) > 2000.0])
	###data['Sale Amount']最好一步步赋值，这样避免出错
	###此处的data['Sale Amount']不可以用str，因为它本身就是str，不能用str再访问（书上例子跑不出）
filtered_rows = pd.concat(row_output, axis=0, ignore_index=True)
writer = pd.ExcelWriter(output_file)
filtered_rows.to_excel(writer, sheet_name='sale_amount_gt2000', index=False)
writer.save()
```

### 在所有工作表中选取特定列
#### 基础python

```
#!/usr/bin/env python3
import sys
from datetime import date
from xlrd import open_workbook, xldate_as_tuple
from xlwt import Workbook
input_file = sys.argv[1]
output_file = sys.argv[2]
output_workbook = Workbook()
output_worksheet = output_workbook.add_sheet('selected_columns_all_worksheets')
my_columns = ['Customer Name', 'Sale Amount']#创建列表变量
first_worksheet = True
with open_workbook(input_file) as workbook:
	data = [my_columns]##将变量放入data，成为data中的第一个列表
	index_of_cols_to_keep = []##保存列中的值
	for worksheet in workbook.sheets():
		if first_worksheet:
			header = worksheet.row_values(0)
			for column_index in range(len(header)):
				if header[column_index] in my_columns:
					index_of_cols_to_keep.append(column_index)
			first_worksheet = False
		for row_index in range(1, worksheet.nrows):
			row_list = []
			for column_index in index_of_cols_to_keep:	
				cell_value = worksheet.cell_value(row_index, column_index)
				cell_type = worksheet.cell_type(row_index, column_index)
				if cell_type == 3:###又是一段处理日期的鬼
					date_cell = xldate_as_tuple(cell_value,workbook.datemode)
					date_cell = date(*date_cell[0:3]).strftime('%m/%d/%Y')
					row_list.append(date_cell)
				else:
					row_list.append(cell_value)
			data.append(row_list)
	for list_index, output_list in enumerate(data):
		for element_index, element in enumerate(output_list):
			output_worksheet.write(list_index, element_index, element)
output_workbook.save(output_file)
```

#### pandas
##### 此处使用了read_excel函数和loc函数
- 即：用read_excel函数将所有工作表读入一个字典
- 用loc函数在每个工作表中选取特定的列，创捷一个筛选过的数据框列表
- 并连接在一起，形成一个最终数据框
```
#!/usr/bin/env python3
import pandas as pd
import sys
input_file = sys.argv[1]
output_file = sys.argv[2]
data_frame = pd.read_excel(input_file, sheet_name=None, index_col=None)
column_output = []
for worksheet_name, data in data_frame.items():
	column_output.append(data.loc[:, ['Customer Name', 'Sale Amount']])
selected_columns = pd.concat(column_output, axis=0, ignore_index=True)
writer = pd.ExcelWriter(output_file)
selected_columns.to_excel(writer, sheet_name='selected_columns_all_worksheets', index=False)
writer.save()
```

## 在excel工作簿中读取一组工作表
- 可以用sheet_by_index或sheet_by_name函数来处理一组工作表
###在一组工作表中筛选特定行 
#### 基础python

```
#!/usr/bin/env python3
import sys
from datetime import date
from xlrd import open_workbook, xldate_as_tuple
from xlwt import Workbook
input_file = sys.argv[1]
output_file = sys.argv[2]
output_workbook = Workbook()
output_worksheet = output_workbook.add_sheet('set_of_worksheets')
my_sheets = [0,1]###来来来，这里又创造了一个列表变量
threshold = 1900.0
sales_column_index = 3
first_worksheet = True
with open_workbook(input_file) as workbook:
	data = []
	for sheet_index in range(workbook.nsheets):
		if sheet_index in my_sheets:##确保处理我们想要处理的那个工作表
			worksheet = workbook.sheet_by_index(sheet_index)#sheet_by_index按索引值来索引sheet
			if first_worksheet:
				header_row = worksheet.row_values(0)
				data.append(header_row)
				first_worksheet = False
			for row_index in range(1,worksheet.nrows):#0行是标题行，已处理。
				row_list = []
				sale_amount = worksheet.cell_value(row_index, sales_column_index)
				if sale_amount > threshold:
					for column_index in range(worksheet.ncols):
						cell_value = worksheet.cell_value(row_index,column_index)
						cell_type = worksheet.cell_type(row_index, column_index)
						if cell_type == 3:
							date_cell = xldate_as_tuple(cell_value,workbook.datemode)
							date_cell = date(*date_cell[0:3]).strftime('%m/%d/%Y')
							row_list.append(date_cell)
						else:
							row_list.append(cell_value)
				if row_list:
					data.append(row_list)
	for list_index, output_list in enumerate(data):
		for element_index, element in enumerate(output_list):
			output_worksheet.write(list_index, element_index, element)
output_workbook.save(output_file)
```
#### pandas
- 没有对比就没有伤害，让我们看一眼简单可爱的pandas
- 我们只需要用read_excel函数中将工作表的索引值或名称设置成一个列表就可以了
```
#!/usr/bin/env python3
import pandas as pd
import sys
input_file = sys.argv[1]
output_file = sys.argv[2]
my_sheets = [0,1]
threshold = 1900.0
data_frame = pd.read_excel(input_file, sheet_name=my_sheets, index_col=None)
row_list = []
for worksheet_name, data in data_frame.items():
	row_list.append(data[data['Sale Amount'].replace('$', '').replace(',', '').astype(float) > threshold])
filtered_rows = pd.concat(row_list, axis=0, ignore_index=True)
writer = pd.ExcelWriter(output_file)
filtered_rows.to_excel(writer, sheet_name='set_of_worksheets', index=False)
writer.save()
```
## 处理多个工作簿
- 要引入glob和os模块，同csv一致
### 工作表计数以及每个工作表中的行列计数
#### python可以做的很漂亮~~~
+ **在不是很熟悉文档情况下，可以通过这种办法，先了解各个工作簿的情况**
```
#!/usr/bin/env python3
import glob
import os
import sys
from xlrd import open_workbook
input_directory = sys.argv[1]
workbook_counter = 0
for input_file in glob.glob(os.path.join(input_directory, '*.xls*')):
	workbook = open_workbook(input_file)
	print('Workbook: {}'.format(os.path.basename(input_file)))
	print('Number of worksheets: {}'.format(workbook.nsheets))
	for worksheet in workbook.sheets():
		print('Worksheet name:', worksheet.name, '\tRows:',worksheet.nrows, '\tColumns:', worksheet.ncols)
	workbook_counter += 1
print('Number of Excel workbooks: {}'.format(workbook_counter))
```
### 从多个工作簿中连接数据
#### 基础python
##### 和csv基本是一样的，但是没有'a'追加模式的问题
- 就是各种读取数据，读取标题行，再把别的行依次输入的过程
```
#!/usr/bin/env python3
import glob #该模块可以定位匹配与某个特定模式的所有路径名
import os #该模块包含了用于解析路径名的函数。
import sys
from datetime import date
from xlrd import open_workbook, xldate_as_tuple
from xlwt import Workbook
input_folder = sys.argv[1]
output_file = sys.argv[2]
output_workbook = Workbook()
output_worksheet = output_workbook.add_sheet('all_data_all_workbooks')
data = []
first_worksheet = True
for input_file in glob.glob(os.path.join(input_folder, '*.xls*')):
	print (os.path.basename(input_file))
	with open_workbook(input_file) as workbook:
		for worksheet in workbook.sheets():
			if first_worksheet:
				header_row = worksheet.row_values(0)
				data.append(header_row)
				first_worksheet = False
			for row_index in range(1,worksheet.nrows):
				row_list = []
				for column_index in range(worksheet.ncols):
					cell_value = worksheet.cell_value(row_index,column_index)
					cell_type = worksheet.cell_type(row_index, column_index)
					if cell_type == 3:
						date_cell = xldate_as_tuple(cell_value,workbook.datemode)
						date_cell = date(*date_cell[0:3]).strftime('%m/%d/%Y')
						row_list.append(date_cell)
					else:
						row_list.append(cell_value)
				data.append(row_list)
for list_index, output_list in enumerate(data):
	for element_index, element in enumerate(output_list):
		output_worksheet.write(list_index, element_index, element)
output_workbook.save(output_file)
```
#### pandas
##### 来来来，concat函数又一次出现了，我们来回顾一下
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
all_workbooks = glob.glob(os.path.join(input_path,'*.xls*'))
data_frames = []
for workbook in all_workbooks:
	all_worksheets = pd.read_excel(workbook, sheet_name=None, index_col=None)
	for worksheet_name, data in all_worksheets.items():
		data_frames.append(data)
all_data_concatenated = pd.concat(data_frames, axis=0, ignore_index=True)
writer = pd.ExcelWriter(output_file)
all_data_concatenated.to_excel(writer, sheet_name='all_data_all_workbooks', index=False)
writer.save()
```

### 为工作簿和工作表计算总数和均值
#### 基础python
##### 这一开头，你是不是就想念sum和mean函数了，罢了罢了，看看不香么
```
#!/usr/bin/env python3
import glob
import os
import sys
from datetime import date
from xlrd import open_workbook, xldate_as_tuple
from xlwt import Workbook
input_folder = sys.argv[1]
output_file = sys.argv[2]
output_workbook = Workbook()
output_worksheet = output_workbook.add_sheet('sums_and_averages')
all_data = []
sales_column_index = 3
header = ['workbook', 'worksheet', 'worksheet_total', 'worksheet_average','workbook_total', 'workbook_average']#徒手插标题
all_data.append(header)
for input_file in glob.glob(os.path.join(input_folder, '*.xls*')):#此处表示一个表一个表的做
	with open_workbook(input_file) as workbook:
		###列表3剑客已经为你建好
		list_of_totals = []
		list_of_numbers = []
		workbook_output = []
		for worksheet in workbook.sheets():
			total_sales = 0
			number_of_sales = 0
			worksheet_list = []#我worksheet_list，用来给你们保存各种信息
			worksheet_list.append(os.path.basename(input_file))
			worksheet_list.append(worksheet.name)
			for row_index in range(1,worksheet.nrows):
				try:
					total_sales += float(str(worksheet.cell_value(row_index,sales_column_index)).strip('$').replace(',',''))
					number_of_sales += 1.
				except:#？？？？？当某行没有销售数据时，忽略该行？？？？
					total_sales += 0.
					number_of_sales += 0.
			average_sales = '%.2f' % (total_sales / number_of_sales)
			worksheet_list.append(total_sales)
			worksheet_list.append(float(average_sales))###如果不加float，输出的excel中的average_sales是str，不是float，不能用于计算啊啥的
			list_of_totals.append(total_sales)
			list_of_numbers.append(float(number_of_sales))
			workbook_output.append(worksheet_list)
		workbook_total = sum(list_of_totals)
		workbook_average = sum(list_of_totals)/sum(list_of_numbers)
		for list_element in workbook_output:
			list_element.append(workbook_total)
			list_element.append(workbook_average)
		all_data.extend(workbook_output)##extend() 函数用于在列表末尾一次性追加另一个序列中的多个值（用新列表扩展原来的列表）。
for list_index, output_list in enumerate(all_data):
	for element_index, element in enumerate(output_list):
		output_worksheet.write(list_index, element_index, element)
output_workbook.save(output_file)
```
#### pandas
- 讲这些数据连接成一个独立数据框，并写入输出文件。
- 乖乖！怎么没有用mean函数啊
？？？？？？大大大大大大大的问号，跑不出来。。。。反正到后面再看
```
#!/usr/bin/env python3
import pandas as pd
import glob
import os
import sys
input_path = sys.argv[1]
output_file = sys.argv[2]
all_workbooks = glob.glob(os.path.join(input_path,'*.xls*'))
data_frames = []
for workbook in all_workbooks:
	all_worksheets = pd.read_excel(workbook, sheet_name=None, index_col=None)
	workbook_total_sales = []
	workbook_number_of_sales = []
	worksheet_data_frames = []
	worksheets_data_frame = None
	workbook_data_frame = None
	for worksheet_name, data in all_worksheets.items():
		total_sales = pd.DataFrame([float(str(value).strip('$').replace(',','')) for value in data.loc[:, 'Sale Amount']]).sum()
		number_of_sales = len(data.loc[:, 'Sale Amount'])
		average_sales = pd.DataFrame(total_sales / number_of_sales)
		workbook_total_sales.append(total_sales)
		workbook_number_of_sales.append(number_of_sales)
		data = {'workbook': os.path.basename(workbook),
				'worksheet': worksheet_name,
				'worksheet_total': total_sales,
				'worksheet_average': average_sales}
		worksheet_data_frames.append(pd.DataFrame(data, columns=['workbook', 'worksheet', 'worksheet_total', 'worksheet_average']))
	worksheets_data_frame = pd.concat(worksheet_data_frames, axis=0, ignore_index=True)
	workbook_total = pd.DataFrame(workbook_total_sales).sum()
	workbook_total_number_of_sales = pd.DataFrame(workbook_number_of_sales).sum()
	workbook_average = pd.DataFrame(workbook_total / workbook_total_number_of_sales)
	workbook_stats = {'workbook': os.path.basename(workbook),
					 'workbook_total': workbook_total,
					 'workbook_average': workbook_average}
	workbook_stats = pd.DataFrame(workbook_stats, columns=['workbook', 'workbook_total', 'workbook_average'])
	workbook_data_frame = pd.merge(worksheets_data_frame, workbook_stats, on='workbook', how='left')
	data_frames.append(workbook_data_frame)
all_data_concatenated = pd.concat(data_frames, axis=0, ignore_index=True)
writer = pd.ExcelWriter(output_file)
all_data_concatenated.to_excel(writer, sheet_name='sums_and_averages', index=False)
writer.save()
```









