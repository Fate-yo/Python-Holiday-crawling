​

这是我们python课程要求我们制作一个项目，用python爬取结果并作数据展示。我们使用requests的方法对房价的信息做了爬取，一下就是我们所爬取的网页 


我们做这个项目主要分为以下几个步骤
1 网页爬取过程
        我们使用类的方法经行了封装在直接输入城市名的时候就可以直接get到数据

```
class reptile:
    def __init__(self):
        self.__city = '天津'
        self.__header = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.55 Safari/537.36 Edg/96.0.1054.43'
        }

    def up_data(self, city):
        if city != '':
            self.__city = city
        else:
            print('没有得到新的城市名。')

    def write_in(self, data, *, fileName='', title='', time=False):
        # 数据写入
        flag = False
        with open(fileName, 'w', encoding='utf-8') as fp:
            if not title == '':
                fp.write(title + '\n')
            if time:
                for i, j in data:
                    if flag:
                        fp.write('\n')
                    else:
                        flag = True
                    fp.write(str(get_really_time(i)) + ':')
                    fp.write(str(j))
            else:
                for i, j in data.items():
                    if flag:
                        fp.write('\n')
                    else:
                        flag = True
                    fp.write(i + ' ')
                    for k in j:
                        fp.write(k + ' ')


    def show_all(self):
        oneyear_m()
        main()

    def get_photo_data(self):  # 获取目标城市的总体价格走势图的数据
        url = 'http://' + get_first(self.__city) + '.fangjia.com/trend/yearData?'
        param = {
            'defaultCityName': self.__city,
            'districtName': '',
            'region': '',
            'block': '',
            'keyword': ''
        }
        res = requests.get(url=url, params=param, headers=self.__header).json()
        data = res['series']
        d = data[0]['data']
        # 文件写入
        self.write_in(d, fileName='zoushi.txt', time=True)

    def get_which(self, choose='up'):
        url = 'http://' + get_first(self.__city) + '.fangjia.com/zoushi'

        page_txt = requests.get(url=url, headers=self.__header).text
        if choose == 'up':
            ex = '<div class="trend trend03">.*?<tbody>(.*?)<tbody>'
        else:
            ex = '<div class="trend trend03" style="border-bottom:none;">.*?<tbody>(.*?)</tbody>'
        url_list = str(re.findall(ex, page_txt, re.S)[0])
        ex = '<tr class=".*?">(.*?)</tr>'
        all = str(re.findall(ex, url_list, re.S))
        ex_name = '<td class="td02"><a href=".*?">(.*?)</a></td>'
        ex_data = '<td>(.*?)</td>'
        need_name = re.findall(ex_name, all, re.S)
        need_data = re.findall(ex_data, all, re.S)
        need_data = [i for i in need_data if not i == '元/㎡' and not i == '周度']
        d = {}
        i = 1
        for house_name in need_name:
            d[house_name] = need_data[4 * (i - 1):4 * i]
            i += 1
        self.write_in(d, fileName='data_' + choose + '.txt')
 ```

2 数据可视化  
         我们主要爬取的内容包括了房价的走势，上月的价格，本月的价格，和历史最高的价格和涨幅，等信息做了爬取并用matplotlib 画出了一个折线图并将其保存下来结果呈现  。以成都为例 爬取成都一年来的房价走势。
```
def oneyear_m():
    x = []
    y = []
    with open("zoushi.txt", 'r', encoding='utf-8') as data1:
        for line in data1.read().split("\n"):
            data1_line = line.split(":")
            x.append(data1_line[0][5:])
            y.append(int(data1_line[1]))
    plt.figure(figsize=(28, 10))
    plt.title('一年变化图')  # 折线图标题
    plt.rcParams['font.sans-serif'] = ['SimHei']  # 显示汉字
    plt.xlabel('时间')  # x轴标题
    plt.ylabel('价格   (元/㎡)')  # y轴标题
    plt.plot(x, y, marker='o', markersize=5)  # 绘制折线图，添加数据点，设置点的大小
    for a, b in zip(x, y):
        plt.text(a, b, b, ha='center', va='bottom', fontsize=10)  # 设置数据标签位置及大小
    plt.legend(['走势'])  # 设置折线名称
    plt.savefig('一年变化图.jpg')
    plt.show()
同时在建立其他几个项目的同时我们直接定义了一个函数将需要的参数传入函数进行画图避免了一个图一个函数导致的代码冗余


def paint(x, y,flag):  # 小区上月价格折线图
    plt.figure(figsize=(10, 5))
    plt.title(flag)  # 折线图标题
    plt.rcParams['font.sans-serif'] = ['SimHei']  # 显示汉字
    plt.xlabel('时间')  # x轴标题
    plt.ylabel('价格   (元/㎡)')  # y轴标题
    plt.plot(x, y, marker='o', markersize=5)  # 绘制折线图，添加数据点，设置点的大小
    for a, b in zip(x, y):
        plt.text(a, b, b, ha='center', va='bottom', fontsize=10)  # 设置数据标签位置及大小
    plt.legend(['方案'])  # 设置折线名称
    plt.savefig(flag+'.jpg')
    plt.show()

 其他的爬取结果展示如下：

```

3. 我们在制作这个项目的时候也遇到了一些常见的问题供大家分享：
        比如我么在分析数据的时候使用了正则表达式，在使用正则表达式的过程中会遇到很多的问题，不是很好处理。在这里可以推荐使用BeautifulSoup的方法会更加简单如果大家需要学习正则表达式推荐一个文章    正则表达式     或者 是学习使用BeautifulSoup的方法这两篇博客都是比较推荐的解析的方法

  还有个问题就是在使用matplotlib画图并保存的时候需要注意  plt.savefig()需要放在
    plt.show()的前面，不然会导致生成出来的图片是空白，我们开始就遇到了这个问题，生成的图片一直是空白，后面才发现更改了之后一下就生成了一张折线图

下面更新了画图的方式
        我们之前使用的matplotlib的方法来画折线图，后来发现不是特别好，于是我们改成了 pygal 库的方法来画图更新画图主题部分如下：
```
def paint(x, y, y1, y2, name1, name2, name3, types):  # 小区上月价格折线图
    line_chart = pygal.Line(x_label_rotation=250, height=250, show_minor_x_labels=True, style=NeonStyle)
    line_chart.title = types + "房价分析"
    line_chart.x_labels = x
    line_chart.x_labels_major = x
    line_chart.add(name1, y)
    line_chart.add(name2, y1)
    line_chart.add(name3, y2)
    line_chart.render_to_file(types + "房价分析" + '.html')
```

