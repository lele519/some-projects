# 应用程序
## 在一个大文件集合中查找一个项目
- 当需要读取的文件量大且excel和csv混合时的解决办法
- 小小的模拟一下
	+ 在桌面新建 file_achive文件夹
	 + 将csv和excel放入文件夹
	 + 最好有：
		+ csv文件：*.csv
		+ excel文件：*.xls（含多个工作表）
		+ excel文件(可选)：*.xlsx（含多个工作表）
- 如果只是搜索很少的数值项目，就可以使用列表或元组变量在python脚本中将条件写死。(eg:items_to_look_for=['1234','2345'])
	- 当项目数成百上千时，该方法则笨重切不可行。
- THUS
- 我们要使用向脚本中传输数据的方法，并将数值项目放在csv输入文件的一列中
	+ 此法可以搜索大量的数值项目，都可以将它们写在csv输入文件中，然后读入python脚本
	 + 该法有很好的扩展性

- 我们要建一个csv文档，将target输入，写在A列，不需要标题行。如果有标题行，脚本中时要去掉的。
- 一直没有跑出来脚本的原因：python 1search_for_items_write_found.py item_numbers_to_find.csv file_achive 1o.csv
	+ 一是执行命令文件夹名字拼错
	 + 二是，需要查询的文件夹路径有问题，最好是和所运行脚本在同一文件夹下，尝试过换路径的办法，但是output没有任何内容？？？？有空自行再调整。
```
#!/usr/bin/env python3
#来来来，路径、全球流、csv、excel、日期等，该import就import。
import csv
import glob#该模块可以定位匹配与某个特定模式的所有路径名
import os
import sys
from datetime import date
from xlrd import open_workbook, xldate_as_tuple##这里是读取excel文档，所以用xlrd
item_numbers_file = sys.argv[1] ##目标数值项目的路径，即target#路径
path_to_folder = sys.argv[2] ##所要搜索的文件所在文件夹路径
output_file = sys.argv[3] ##输出路径
item_numbers_to_find = []
with open(item_numbers_file, 'r', newline='') as item_numbers_csv_file:
	filereader = csv.reader(item_numbers_csv_file)
	for row in filereader:
		item_numbers_to_find.append(row[0])##加入行0号位，即列一
#print(item_numbers_to_find)
	filewriter = csv.writer(open(output_file, 'a', newline=''))###来来来，睁大眼睛看啊，是追加模式呢！！！
file_counter = 0##读入脚本的历史文件数量
line_counter = 0###所有输入文件和工作表中读出的行
count_of_item_numbers = 0####行中数值项目是我们想要搜索的数值项目数
for input_file in glob.glob(os.path.join(path_to_folder, '*.*')):#在path_to_folder路径中，找到*.*匹配的文件
	###*.*：表任意开头的,_后面可以是任意字符的文件。
	file_counter += 1
	if input_file.split('.')[1] == 'csv':#如果是csv文件，打开csv文件
		with open(input_file, 'r', newline='') as csv_in_file:
			filereader = csv.reader(csv_in_file)
			header = next(filereader)
			for row in filereader:
				row_of_output = []
				for column in range(len(header)):##读每一列
					if column < 3:##Item Number、Description、Supplier这三项没有特殊字符，主要是英文和数字，格式化为str即可
						##其实可以省略，因为和else的命令一样，万一改date可以再设置一个
						cell_value = str(row[column]).strip()
						row_of_output.append(cell_value)
					elif column == 3:##cost项会有$和,所以要多几个步骤保证数额的准确
						cell_value = str(row[column]).lstrip('$').replace(',','').split('.')[0].strip()##split('.')[0]按.进行分割[0]表句点前，[1]表句点后。
						###并没有感受到此处split('.')[0]的必须性
						row_of_output.append(cell_value)
					else:##剩余的值的处理
						cell_value = str(row[column]).strip()
						row_of_output.append(cell_value)
				row_of_output.append(os.path.basename(input_file))##os.path.basename(input_file)函数：表从完整路径名中抽取基本文件名
				###这个input_file中包含的字符串是file_achive\supplies_2012.csv
				###os.path.basename(input_file)函数确保代码只将supplies_2012.csv追加到表row_of_output
				if row[0] in item_numbers_to_find:#判断每行的0号位是不是在item_numbers_to_find
					filewriter.writerow(row_of_output)
					count_of_item_numbers += 1###用来计数所有文件找到数值项目的数量
				line_counter += 1###耿总所有输入文件中找到的数据行的数量
	elif input_file.split('.')[1] == 'xls' or input_file.split('.')[1] == 'xlsx':##hey！这里打开excel文件啦！
		workbook = open_workbook(input_file)###用open_workbook()打开input_file并赋给workbook
		for worksheet in workbook.sheets():
			try:
				header = worksheet.row_values(0)
			except IndexError:###如果出现IndexError就说明下表越界，然后pass，即什么都不做
				pass
			for row in range(1, worksheet.nrows):###跳过标题行开始做循环
				row_of_output = []
				for column in range(len(header)):#读每一列，操作和csv一样，不过用的是excel的方法
					if column < 3:
						cell_value = str(worksheet.cell_value(row,column)).strip()
						row_of_output.append(cell_value)
					elif column == 3:
						cell_value = str(worksheet.cell_value(row,column)).split('.')[0].strip()
						row_of_output.append(cell_value)
					else:##此处特地处理了date
						cell_value = xldate_as_tuple(worksheet.cell(row,column).value,workbook.datemode)
				###这个单元格的值作为xldate_as_tuple函数中的一个参数，会被转换成元组中的一个代表日期的浮点数
				###workbook.datemode是**必须的**，它可以使函数确定日期是基于1900还是1904（mac上有的版本是基于1904的）
				###workbook.datemode函数的结果被赋给一个元组变量cell_value
						cell_value = str(date(*cell_value[0:3])).strip()##此处的*必须存在，不然报错：TypeError: an integer is required (got type tuple)
						###个人猜想：*在此表示参数？是*args的意思？
						###cell_value[0:3]：使用元组索引，引用元组cell_value中的前3个元素（即年月日）
						row_of_output.append(cell_value)
				row_of_output.append(os.path.basename(input_file))
				row_of_output.append(worksheet.name)##因为excel中含多个工作表，所以此行要将工作表名称追加到列表中。清楚的知道数值项目来源
				if str(worksheet.cell(row,0).value).split('.')[0].strip() in item_numbers_to_find:
					filewriter.writerow(row_of_output)
					count_of_item_numbers += 1
				line_counter += 1
print('Number of files: {}'.format(file_counter))
print('Number of lines: {}'.format(line_counter))
print('Number of item numbers: {}'.format(count_of_item_numbers))
```
- 输出中的日期格式不统一，调试代码没有成功，等复习时再次调试一下excel的date格式，按理是直接strftime('%m/%d/%Y')。？？思考
	+ 但是上述代码格式化了源文件的日期格式，所以，emmmm.....

## 为csv文件中数据的任意数目分类计算统计量
- ？？？做数据周期性分析？？回头客分析？？？
- 本次案例：客户买了个某项基本服务后，多久会升级
	+ 先建立一个customer_category_history.csv的数据集
- **!**这是第一个用python字典数据结构来组织和保存计算结果的实例哦~~
	+ 还是个嵌套字典。。。。
	+ 外部字典名称：packages{key：客户名称，value：}，与该key相对的另一个字典{key：服务包的名称，value:整数（客户拥有这个服务包的天数）}
```
#!/usr/bin/env python3
import csv
#这里用csv大概率是db的output用csv比较节省空间？
import sys
from datetime import date, datetime
def date_diff(date1, date2):###定义一个date_diff的函数
	try:
		diff = str(datetime.strptime(date1, '%m/%d/%Y') - datetime.strptime(date2, '%m/%d/%Y')).split()[0]
	##用datetime.strptime（）按照日期字符创建datetime对象，并将结果str。
	##split()[0]：按空格分割，保留字符串分割后的0号位（即最左部分），并赋值给diff
	except:##忽略特殊情况：diff = 0
		diff = 0
	if diff == '0:00:00':
		diff = 0
	return diff
input_file = sys.argv[1]
output_file = sys.argv[2]
packages = {}
previous_name = 'N/A'
previous_package = 'N/A'
previous_package_date = 'N/A'
###将字符串'N/A'赋值给这三个变量，前提是'N/A'一定不会出现在文件各列中的字符串。
first_row = True##用来后面处理第一行的数据
today = date.today().strftime('%m/%d/%Y')#取了今天的日子，顺手给他洗了
with open(input_file, 'r', newline='') as input_csv_file:
	filereader = csv.reader(input_csv_file)
	header = next(filereader)
	for row in filereader:###一行一行跑处理，跑判断。因为excel的文件已经将客户按名字拍好序，所以循环完一个客户，循环另一个。
		###先将一个客户的一个服务的服务时间不断的循环加给diff
		###当客户名称、服务出现改变，则字典也出现改变
		current_name = row[0]###取客户名
		current_package = row[1]##取服务
		current_package_date = row[3]##取时间
		if current_name not in packages:#如果current_name不在packages这个字典中
			packages[current_name] = {}##将current_name中的值作为字典的key加入到字典packages中，并将这个key对应的value设为空
		if current_package not in packages[current_name]:#检验current_packages变量中的值是不是内部字典中的一个key，
			packages[current_name][current_package] = 0##如果不是就将current_packages作为字典的key加入字典 packages，将这个可以对应的value设为0
			###此时第一行输入完字典应该长这样{'John Smith:{'Bronze':0}}
		if current_name != previous_name:##因为John Smith！= 'N/A'我们进入if...else
			if first_row:###第一行豁免大法
				first_row = False
			else:
				diff = date_diff(today, previous_package_date)##使用自定义的date_diff函数计算diff
				if previous_package not in packages[previous_name]:
					packages[previous_name][previous_package] = int(diff)
				else:
					packages[previous_name][previous_package] += int(diff)
		else:
			diff = date_diff(current_package_date, previous_package_date)
			packages[previous_name][previous_package] += int(diff)
		previous_name = current_name
		previous_package = current_package
		previous_package_date = current_package_date
header = ['Customer Name', 'Category', 'Total Time (in Days)']###空手插title
with open(output_file, 'w', newline='') as output_csv_file:
    filewriter = csv.writer(output_csv_file)
    filewriter.writerow(header)
    for customer_name, customer_name_value in packages.items():
        for package_category, package_category_value in packages[customer_name].items():
            row_of_output = []
            print(customer_name, package_category, package_category_value)
            row_of_output.append(customer_name)
            row_of_output.append(package_category)
            row_of_output.append(package_category_value)
            filewriter.writerow(row_of_output)
filewriter = csv.writer(output_csv_file)###该行可以和两个for中任意一个对齐，但是求求你别在放进最后一个循环里，跑不出来啊啊！！！！！
```
## 为文本文件中数据的任意数目分类计算统计量
- 该章节的文件是**文本文件**
	+ 文本文件（也称平面文件），也是商业中常用文件类型。
	 + csv实际上就是以逗号分隔的文本文件形式保存的。
	 + 活动日志、错误日志和交易记录是商业数据保存在文本文件上的几个常见例子。
- 此案例是拿错误日志文件为案本
- 给当日的给错误计数
```
#!/usr/bin/env python3
import sys
###因为是纯文本文件，所以不需要导入csv或xlrd模块
input_file = sys.argv[1]
output_file = sys.argv[2]
messages = {}##它也是给嵌套字典{key：错误发生的具体日期，value：{key：错误消息，value：某一天错误发生的次数}}
notes = []###保存输入的错误日志文件中所有日期发生的全部错误消息。将所有错误消息放在一个独立的数据结构中（放在一个列表，而不是前面的字典）。
##为了是可以更容易地在错误文件中搜索错误消息，将错误消息作为标题写入输出文件，并在字典和列表中分别进行迭代，将日期和数据计数写在输出文件中。
with open(input_file, 'r', newline='') as text_file:##老规矩，读个文件先
	for row in text_file:
		if '[Note]' in row:##含字符串[Note]就是包含错误信息的行
			row_list = row.split(' ', 4)##用空格最多拆分4次，将五个部分赋给row_list
			###通过文本文件我们可以看到，前四个空格分隔出来了4个不同的片段，第五部分则是错误内容
			day = row_list[0].strip()#0号位是日期
			note = row_list[4].strip('\n').strip()#将第五部分即位号4（错误信息）的部分赋给note
			##从下面开始就是读数，增加值，计数的过程
			if note not in notes:
				notes.append(note)
			if day not in messages:
				messages[day] = {}
			if note not in messages[day]:
				messages[day][note] = 1
			else:
				messages[day][note] += 1
filewriter = open(output_file, 'w', newline='')
header = ['Date']
header.extend(notes)##将错误信息往后加
header = ','.join(map(str,header)) + '\n'###使用map()、join()和str()函数将列表变量header中的内容在写入输出文件之间转换成一个长字符串
##将header作为csv输出的第一行
print(header)
filewriter.write(header)
for day, day_value in messages.items():
	row_of_output = []
	row_of_output.append(day)	
	for index in range(len(notes)):
		if notes[index] in day_value.keys():
			row_of_output.append(day_value[notes[index]])
		else:
			row_of_output.append(0)
	output = ','.join(map(str,row_of_output)) + '\n'###使用map()、join()和str()函数将列表变量header中的内容在写入输出文件之间转换成一个长字符串
	print(output)
	filewriter.write(output)
filewriter.close()
```
- 给当日各个时间的错误计数
```
#!/usr/bin/python
import string
import sys
input_file = sys.argv[1]
output_file = sys.argv[2]
messages = {}
notes = []
with open(input_file, 'r', newline='') as text_file:
	for row in text_file:
		if '[Note]' in row:
			n = 2
			groups = row.split(' ')
			date_time = ' '.join(groups[:n])
			rest_of_line_string = ' '.join(groups[n:])
			rest_of_line_list = rest_of_line_string.split(' ', 2)
			note = rest_of_line_list[2].strip('\n').strip()
			row_list = []
			row_list.append(date_time)
			row_list.append(note)
			print (row_list)
			day = row_list[0]
			note = row_list[1]
			if note not in notes:
				notes.append(note)
			if day not in messages:
				messages[day] = {}
			if note not in messages[day]:
				messages[day][note] = 1
			else:
				messages[day][note] += 1
filewriter = open(output_file, 'w', newline='')
header = ['Date']
header.extend(notes)
header = ','.join(map(str,header)) + '\n'
print(header)
filewriter.write(header)
for day, day_value in messages.items():
	row_of_output = []
	row_of_output.append(day)	
	for index in range(len(notes)):
		if notes[index] in day_value.keys():
			row_of_output.append(day_value[notes[index]])
		else:
			row_of_output.append(0)
	output = ','.join(map(str,row_of_output)) + '\n'
	print(output)
	filewriter.write(output)
filewriter.close()
```






































