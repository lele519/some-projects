# 图与图表
- python有许多绘图扩展包
## matplotlib
- matplotlib是最基础的扩展包，为pandas和seaborn提供了一些基础的绘图概念和语法
- 其创建的图形可以达到出版的质量要求
- 可以参考matplotlib初学者指南和[http://matplotlib.org/users/beginner.html](API)
### 条形图
```
#!/usr/bin/env python3
import matplotlib.pyplot as plt##啥话不说import一个包
plt.style.use('ggplot')#用ggplot样式来表来模拟ggplot2风格的图形，ggplot2是一个常用的R语言绘图包
customers = ['ABC', 'DEF', 'GHI', 'JKL', 'MNO']#设置数据
customers_index = range(len(customers))
sale_amounts = [127, 90, 201, 111, 232]#设置数据
fig = plt.figure()#先建一个基础图【是个光板】
ax1 = fig.add_subplot(1,1,1)##向基础图添加一个子图，(1,1,1)：一行一列一个子图。
ax1.bar(customers_index, sale_amounts, align='center', color='darkblue')##创建条形图。customers_index：x轴的坐标。sale_amounts：设置条形高度。
###align='center'：与标签中间对齐，color='darkblue'：颜色为深蓝。
ax1.xaxis.set_ticks_position('bottom')#设置x轴刻线位子在底部
ax1.yaxis.set_ticks_position('left')#设置y轴刻线位置在左侧
plt.xticks(customers_index, customers, rotation=0, fontsize='small')##将条形的刻度先标签由客户索引值更改为实际客户名称
###rotation=0：刻度线应该是水平的，fontsize='small'：将刻度标签的字体设为小字体
plt.xlabel('Customer Name')
plt.ylabel('Sale Amount')
plt.title('Sale Amount per Customer')
plt.savefig('bar_plot.png', dpi=400, bbox_inches='tight')#将图保存在当前运行脚本的文件夹中，dpi=400：设置图形分辨率 bbox_inches='tight'：保存图形时把四周空白去掉
plt.show()
```
### 直方图
```
#!/usr/bin/env python3
import numpy as np
import matplotlib.pyplot as plt
plt.style.use('ggplot')#用ggplot样式来表来模拟ggplot2风格的图形，ggplot2是一个常用的R语言绘图包
mu1, mu2, sigma = 100, 130, 15
##随机创建两个正太分布x1、x2
##因为mu1 ！= mu2所以两个图不会覆盖，但会有部分重叠
x1 = mu1 + sigma*np.random.randn(10000)
x2 = mu2 + sigma*np.random.randn(10000)
fig = plt.figure()#先建一个基础图【是个光板】
ax1 = fig.add_subplot(1,1,1)##向基础图添加一个子图，(1,1,1)：一行一列一个子图。
n, bins, patches = ax1.hist(x1, bins=50, density=False, color='darkgreen')##bins=50：每个变量的值被分成50份，density=False：直方图显示的时频率分布
n, bins, patches = ax1.hist(x2, bins=50, density=False, color='orange', alpha=0.5)##alpha=0.5：表示该图应是透明的
ax1.xaxis.set_ticks_position('bottom')#设置x轴刻线位子在底部
ax1.yaxis.set_ticks_position('left')#设置y轴刻线位置在左侧
plt.xlabel('Bins')
plt.ylabel('Number of Values in Bin')
fig.suptitle('Histograms', fontsize=14, fontweight='bold')##设置了一个居中标题，字体大小14，粗体
ax1.set_title('Two Frequency Distributions')##为子图添加一个剧中标题
plt.savefig('histogram.png', dpi=400, bbox_inches='tight')
plt.show()
```
### 折线图
```
#!/usr/bin/env python3
from numpy.random import randn
import matplotlib.pyplot as plt
plt.style.use('ggplot')#用ggplot样式来表来模拟ggplot2风格的图形，ggplot2是一个常用的R语言绘图包
##用randn创建绘图所用的随机数据
plot_data1 = randn(50).cumsum()
plot_data2 = randn(50).cumsum()
plot_data3 = randn(50).cumsum()
plot_data4 = randn(50).cumsum()
fig = plt.figure()
ax1 = fig.add_subplot(1,1,1)
ax1.plot(plot_data1, marker=r'o', color='blue', linestyle='-', label='Blue Solid')
ax1.plot(plot_data2, marker=r'+', color='red', linestyle='--', label='Red Dashed')
ax1.plot(plot_data3, marker=r'*', color='green', linestyle='-.', label='Green Dash Dot')
ax1.plot(plot_data4, marker=r's', color='orange', linestyle=':', label='Orange Dotted')
ax1.xaxis.set_ticks_position('bottom')
ax1.yaxis.set_ticks_position('left')
ax1.set_title('Line Plots: Markers, Colors, and Linestyles')
plt.xlabel('Draw')
plt.ylabel('Random Number')
plt.legend(loc='best')##loc='best':根据图中空白部分将图放在最合适的位置
plt.savefig('line_plot.png', dpi=400, bbox_inches='tight')
plt.show()
```
### 散点图
```
#!/usr/bin/env python3
import numpy as np###抽空看看numpy
import matplotlib.pyplot as plt
plt.style.use('ggplot')
x = np.arange(start=1., stop=15., step=1.)##从1开始到15，间隔为1
y_linear = x + 5. * np.random.randn(14) ##直线方程，取随机数，使和原来的偏离**np.random.randn(int)**
y_quadratic = x**2 + 10. * np.random.randn(14)##二次曲线方程，取随机数，使和原来的偏离
##使用numpy的polyfit函数通过两组数据点(x, y_linear, deg=1)和(x, y_quadratic, deg=2)拟合出一条直线和一条二次曲线
##再使用poly1d（）函数根据直线和二次曲线的参数生成一个线性方程和一个二次方程
fn_linear = np.poly1d(np.polyfit(x, y_linear, deg=1))
fn_quadratic = np.poly1d(np.polyfit(x, y_quadratic, deg=2))
fig = plt.figure()
ax1 = fig.add_subplot(1,1,1)
ax1.plot(x, y_linear, 'bo', x, y_quadratic, 'go', x, fn_linear(x), 'b-', x, fn_quadratic(x), 'g-', linewidth=2.)
##(x, y_linear)表现为'bo',(x, y_quadratic)表现为'go', (x, fn_linear(x))表现为'b-',( x, fn_quadratic(x))表现为'g-', linewidth设置线的宽度
ax1.xaxis.set_ticks_position('bottom')
ax1.yaxis.set_ticks_position('left')
ax1.set_title('Scatter Plots with Best Fit Lines')
plt.xlabel('x')
plt.ylabel('f(x)')
plt.xlim((min(x)-1., max(x)+1.))##设置x轴的范围
plt.ylim((min(y_quadratic)-10., max(y_quadratic)+10.))##设置y轴的范围
plt.savefig('scatter_plot.png', dpi=400, bbox_inches='tight')
plt.show()
```
### 箱线图
- 最小值、第一四分位数、中位数、第三四分位数、最大值
```
#!/usr/bin/env python3
import numpy as np
import matplotlib.pyplot as plt
plt.style.use('ggplot')
N = 500
normal = np.random.normal(loc=0.0, scale=1.0, size=N)
lognormal = np.random.lognormal(mean=0.0, sigma=1.0, size=N)
index_value = np.random.randint(1,N-1,size=N)##random_integers()该函数已被弃用
normal_sample = normal[index_value]
lognormal_sample = lognormal[index_value]
box_plot_data = [normal,normal_sample,lognormal,lognormal_sample]
fig = plt.figure()
ax1 = fig.add_subplot(1,1,1)
box_labels = ['normal','normal_sample','lognormal','lognormal_sample']
ax1.boxplot(box_plot_data, notch=False, sym='.', vert=True, whis=1.5,showmeans=True, labels=box_labels)##创建四个箱线图
##notch=False：箱体是矩形，而不是中间收缩,sym='.'表离群点使用圆点，而不是默认的+。vert=True：表示箱体是垂直的，不是水平的。
##whis=1.5：设定了只想从第一四分位数和第三四分位数延伸出的范围？？？？.showmeans=True：表示箱体在显示中位数的同时也显示均值
###labels=box_labels：用box_labels来标记箱线图
ax1.xaxis.set_ticks_position('bottom')
ax1.yaxis.set_ticks_position('left')
ax1.set_title('Box Plots: Resampling of Two Distributions')
ax1.set_xlabel('Distribution')
ax1.set_ylabel('Value')
plt.savefig('box_plot.png', dpi=400, bbox_inches='tight')
plt.show()
```
## pandas
- pandas通过提供一个可以用于序列和数据框的函数plot简化了序列和数据框中的数据创建图标过程。
	+ plot函数默认创建折线图，可以通过设置参数kind创建其他类型的图表
### 条形图和箱线图（举一个小例子）
```
#!/usr/bin/env python3
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
plt.style.use('ggplot')
fig, axes = plt.subplots(nrows=1, ncols=2)##创建了一个基础图，和两个并排放置的子图
ax1, ax2 = axes.ravel()##使用ravel()将两个子图值分别赋给ax1和ax2，这样我们就不必使用行和列的索引（即，axes[0,0]和axes[0,1]）来引用子图了
data_frame = pd.DataFrame(np.random.rand(5, 3),##np.random.rand（）当函数括号内有两个及以上参数时，则返回对应维度的数组，能表示向量或矩阵
						index=['Customer 1', 'Customer 2', 'Customer 3', 'Customer 4', 'Customer 5'],
						columns=pd.Index(['Metric 1', 'Metric 2', 'Metric 3'], name='Metrics'))
data_frame.plot(kind='bar', ax=ax1, alpha=0.75, title='Bar Plot')###用plot（）函数建了一个条形图在左侧子图
plt.setp(ax1.get_xticklabels(), rotation=45, fontsize=10)#设置x轴,rotation:旋转角度
plt.setp(ax1.get_yticklabels(), rotation=0, fontsize=10)#设置y轴
ax1.set_xlabel('Customer')
ax1.set_ylabel('Value')
ax1.xaxis.set_ticks_position('bottom')
ax1.yaxis.set_ticks_position('left')
colors = dict(boxes='DarkBlue', whiskers='Gray', medians='Red', caps='Black')#设置了一个颜色字典
data_frame.plot(kind='box', color=colors, sym='r.', ax=ax2, title='Box Plot')#该行创建了一个箱线图在右边子图，颜色随缘，sym='r.'：离群点的形状是红色圆点
plt.setp(ax2.get_xticklabels(), rotation=45, fontsize=10)
plt.setp(ax2.get_yticklabels(), rotation=0, fontsize=10)
ax2.set_xlabel('Metric')
ax2.set_ylabel('Value')
ax2.xaxis.set_ticks_position('bottom')
ax2.yaxis.set_ticks_position('left')
plt.savefig('pandas_plots.png', dpi=400, bbox_inches='tight')
plt.show()
```
## ggplot
- ggplot是基于R语言的ggplot2包和图形语法。
- ggplot和其他绘图包关键区别是它的语法将数据与实际绘图明确地分离开来
- 缺点明显，最好别用，看看拉到
```
#!/usr/bin/env python3
from ggplot import *
print(mtcars.head())
plt1 = ggplot(aes(x='mpg'), data=mtcars) + geom_histogram(fill='darkblue', binwidth=2) + xlim(10, 35) + ylim(0, 10) + \
		xlab("MPG") + ylab("Frequency") + ggtitle("Histogram of MPG") + theme_matplotlib()
print(plt1)
print(meat.head())
plt2 = ggplot(aes(x='date', y='beef'), data=meat) +\
		geom_line(color='purple', size=1.5, alpha=0.75) +\
		stat_smooth(colour='blue', size=2.0, span=0.15) +\
		xlab("Year") + ylab("Head of Cattle Slaughtered") +\
		ggtitle("Beef Consumption Over Time") +\
		theme_seaborn()
print(plt2)
print(diamonds.head())
plt3 = ggplot(diamonds, aes(x='carat', y='price', colour='cut')) +\
		geom_point(alpha=0.5) +\
		scale_color_gradient(low='#05D9F6', high='#5011D1') +\
		xlim(0, 6) + ylim(0, 20000) +\
		xlab("Carat") + ylab("Price") +\
		ggtitle("Diamond Price by Carat and Cut") +\
		theme_gray()
print(plt3)
ggsave(plt3, "ggplot_plots.png")
```
+ 跑不出来，爱谁谁
## seaborn
- seaborn简化了在python中创建信息丰富的统计图表的过程，是在matplotlib基础上开发的，支持numpy和pandas中的数据结构，并继承了spicy和statsmodels中的统计程序
		+ 你简直就是数据分析界的小可爱
- 来来来，我们一个一个跑
- 直方图
```
#!/usr/bin/env python3
import seaborn as sns
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
# Histogram
x = np.random.normal(size=1000)
sns.distplot(x, bins=20, kde=True, rug=False, label="Histogram w/o Density")
sns.utils.axlabel("Value", "Frequency")
plt.title("Histogram of a Random Sample from a Normal Distribution")
plt.legend()
plt.show()
```
- 简单的三个方程的线图
```
#!/usr/bin/env python3
import seaborn as sns
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
sns.set(color_codes=True)
# Simple plot of linear, quadratic, and cubic curves
x = np.linspace(0, 2, 100)
plt.plot(x, x, label='linear')
plt.plot(x, x**2, label='quadratic')
plt.plot(x, x**3, label='cubic')
plt.xlabel('x label')
plt.ylabel('y label')
plt.title("Simple Plot")
plt.legend(loc="best")
plt.show()
```
- Scatter plot
```
#!/usr/bin/env python3
import seaborn as sns
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
sns.set(color_codes=True)
# Scatter plot
mean, cov = [5, 10], [(1, .5), (.5, 1)]
data = np.random.multivariate_normal(mean, cov, 200)
data_frame = pd.DataFrame(data, columns=["x", "y"])
sns.jointplot(x="x", y="y", data=data_frame, kind="reg").set_axis_labels("x", "y")
plt.suptitle("Joint Plot of Two Variables with Bivariate and Univariate Graphs")
plt.show()
```
- Linear regression model
```
#!/usr/bin/env python3
import seaborn as sns
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
#Linear regression model
tips = sns.load_dataset("tips")
sns.lmplot(x="total_bill", y="tip", data=tips)
sns.lmplot(x="size", y="tip", data=tips, x_jitter=.15, ci=None)
sns.lmplot(x="size", y="tip", data=tips, x_estimator=np.mean, ci=None)
plt.show()
```
- # Bar plots
```
#!/usr/bin/env python3
import seaborn as sns
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
sns.set(color_codes=True)
Bar plots
titanic = sns.load_dataset("titanic")
sns.barplot(x="sex", y="survived", hue="class", data=titanic)
sns.countplot(y="deck", hue="class", data=titanic, palette="Greens_d")
plt.show()
```
- Non-linear regression model
```
#!/usr/bin/env python3
import seaborn as sns
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
sns.set(color_codes=True)
#Non-linear regression model
anscombe = sns.load_dataset("anscombe")
sns.lmplot(x="x", y="y", data=anscombe.query("dataset == 'II'"), order=2, ci=False, scatter_kws={"s": 80})
plt.show()
```













