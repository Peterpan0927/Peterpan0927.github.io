---
title: jupyter笔记本基本使用
date: 2017-05-18 16:27:22
tags: 交互式笔记本
categories: Ipython 图表化
---
jupyter笔记本使用（爬虫）
<!--more-->
# jupyter笔记本使用

## 1.在交互式笔记本中进行数据查找

> 导入模块和数据库
>
> ```python
> import pymongo
> client = pymongo.MongoClient('localhost', 27017)
> ganji = client['ganji']
> item_info = ganji['item_info']
> ```
>
> 对数据库进行查询
>
> ```python
> for i in item_info.find().limit(300):
> 	print(i['area'])
> ```
>
> 将area为None的地区替换为不明,去掉标点符号
>
> ```python
> #先导入一个新模块
> from string import punctuation
> #索引的代码优化
> for i in item_info.find():
>     if i['area']:
>         area = [i for i in i['area'] if i not in punctuation]
>     else:
>         area = ['不明']
>     print(area)
> ```
>
> 使用find函数进行精确的查找：
>
> ```python
> import pymongo
> client = pymongo.MongoClient('localhost', 27017)
> ganji = client['ganji']
> item_info = ganji['item_info']
> #第一个大括号内填写查找的条件，条件为空则是全部的数据，后面一个大括号内填写显示的条件，value为0表示不显示,$slice进行分片，返回可被分片数据中的第一个。
> #注意，如果在属性的查找中加入具体的操作符，就会显示其他的属性，这是mongodb的特殊显示方式。
> for i in item_info.find({},{'area':{'$slice:1'},'_id':0,'price':0})
> #$in操作符表示只要包含两个中的一个即可
> for i in item_info.find({'pub_date':{'$in':[2016,2017]}},{'area':{'$slice:1'},'_id':0,'price':0})
> ```
>
> 

## 2.数据更新

```python
#将area为None的替换为不明
for i in item_info.find():
    if i['area']:
        area = [i for i in i['area'] if i not in punctuation]
    else:
        area = ['不明']
    item_info.update_one({'_id':i['_id']}, {'$set':{'area':area}})
```

## 3.利用图表来进行数据的展示

> 环境准备：
>
> ```shell
> pip3 install charts
> #由于版本问题，pip3获取的是对于python2.*版本兼容的模块包，所以我们需要cd到charts的目录下面进行版本更新，不然会疯狂的报错
> 2to3 -w *.py
> #这个命令是使本目录下所有的py文件更新到3.*版本
> ```
>
> 代码部分，[yield参考](http://www.cnblogs.com/tqsummer/archive/2010/12/27/1917927.html)：
>
> ```python
> #统计地区的名字,set用于列表去重
> area_list = []
> for i in item_info.find():
>     area_list.append(i['area'][0])
> area_index = list(set(area_list))
> #计算出现的次数
> post_times = []
> for index in area_index:
>     post_times.append(area_list.count(index))
> #将数据格式转换成可视化图表模式的代码
> def data_gen(types):
>     length = 0
>     if length <= len(area_index):
>     	for area,times in zip(area_index,post_times):
>         	data = {
>           	'name' : area,
>           	'data' : [times],
>           	'type' : types
>         	}
>         	yield data
>         	length += 1
> #最后生成图表
> series = [data for data in data_gen('column')]
> charts.plot(series, show='inline', options=dict(title=dict(text='七日内二手物品销售统计')))
> ```
>
> ![](http://omg5mjb8v.bkt.clouddn.com/252E2171-55DD-407C-82B7-A9BAA6F7E82D.png)

## 4.进行数据的格式修改并更新

```python
#e.g:将‘-’替换为‘.‘
for i in item_info.find():
    frags = i[pub_date].split('-')
    if len(frags) == 1:
        date = frags
    else:
        date = {}.{}.{}.format(frags[1],frags[2],frags[3])
    item_info.update({'_id':i['_id']},{'$set':{'pub_date':date}})5.date函数的使用
```

## 5.使用date函数制作图表[datetime模块](https://my.oschina.net/whp/blog/130710)

```python
#打印一段时间内的连续日期样例
def get_all_dates(date1,date2):
    the_date = date(int(date1.split('.')[0]),int(date1.split('.')[1]),int(date1.split('.')[2]))
    end_date = date(int(date2.split('.')[0]),int(date2.split('.')[1]),int(date2.split('.')[2]))
    days = timedelta(days=1)
    while the_date <= end_date:
            yield the_date.strftime('%Y-%m-%d')
            the_date = the_date + days      
```

```python
#制作图表部分的代码
def get_data_within(date1,date2,areas):
    for area in areas:
        area_day_posts = []
        for day in get_all_dates(date1,date2):
            a = list(item_info.find({'pub_date':date,'area':area}))
            each_posts = len(a)
            area_day_posts.append(each_posts)
        data = {
            'name' : area,
            'data' : area_day_posts,
            'type' : 'lines'
        }
        yield data
```

## 6.使用聚合管道高效查找数据[MongoDB聚合管道](http://blog.csdn.net/zhangzhebjut/article/details/16848045)

![常用的几种](http://omg5mjb8v.bkt.clouddn.com/C4DFB493-E981-4C81-BEA9-DE3F088943A2.png)

```python
#操作符的含义可以参考上面的链接,注意$group的操作是在内存中进行的，所以不能用它来对大量个数的文档来进行分组
pipeline = [
  {'$match':{'$and':[{'pub_date':'2015.4.12'},{'time':3}]}},
  {'$group':{'_id':'$price','counts':{'$sum':1}}},
  {'$sort':{'counts':-1}},
  {'$limit':3}
]
for i in item_info.aggregate(pipeline):
    print(i)
#$slice如果只有一个参数，代表返回几个元素，如果有两个参数，第一个参数代表跳过几个元素，第二个参数代表返回几个元素。
def data_gen(data, time):
    pipeline = [
  	{'$match':{'$and':[{'pub_date':'2015.4.12'},{'time':3}]}},
  	{'$group':{'_id':{'$slice':['$cates':2,1]},'counts':		{'$sum':1}}},
  	{'sort':{'counts':-1}}
	]
    for i in item_info.aggregate(pipeline):
    	yield [i['_id'][0], i['counts']]
#制作饼图
options = {
  'chart' : {'zoomType':'xy'},
  'title' : {'text':'发帖量统计'},
  'subtitle' : {'text':'可视化统计图表'}
}
series = [{
  	'type':'pie'
    'name':'pipe charts',
    'date':[i for i in date_gen('2016.01.10, 1')]
    }]
charts.plot(series, options=options, show='inline')
```

## 7.任务代码

```python
import charts
import pymongo

client = pymongo.MongoClient('localhost', 27017)
ceshi = client['ceshi']
item_info = ceshi['item_infoS']

def data_gen(date1,date2,area,limit):
    pipeline1 = [
    {'$match':{'$and':[{'pub_date':{'$gte':date1,'$lte':date2}},{'area':{'$all':area}}]}},
    {'$group':{'_id':{'$slice':['$cates',2,1]},'counts':{'$sum':1}}},
    {'$limit':limit},
    {'$sort':{'counts':-1}}
]

    for i in item_info.aggregate(pipeline1):
        data = {
            'name': i['_id'][0],
            'data': [i['counts']],
            'type': 'column'
        }
        yield data
series = [i for i in data_gen('2015.12.25','2015.12.27',['朝阳'],5)]
options = {
    'chart'   : {'zoomType':'xy'},
    'title'   : {'text': '发帖数量最大的类目'},
    'subtitle': {'text': '数据图表'},
    'yAxis'   : {'title': {'text': '数量'}}
    }

charts.plot(series,options=options,show='inline')
```

![](http://omg5mjb8v.bkt.clouddn.com/9361A126-4FA8-42FF-8811-A3B6D70ABAFE.png)

```python
#另一种charts的调用方式
def data_gen2(date1,date2,cates):
    pipeline = [
    {'$match':{'$and':[{'pub_date':{'$gte':date1,'$lte':date2}},
                       {'cates':{'$all':cates}},
                       {'look':{'$nin':['-']}}
                      ]}},
    {'$group':{'_id':'$look','avg_price':{'$avg':'$price'}}},
    {'$sort':{'avg_price':1}}
]
    for i in item_info.aggregate(pipeline):
        yield i['avg_price']
data = [i for i in data_gen2('2015.12.24','2016.01.10',['北京二手手机'])]
options = {
    'title': {'text': '新旧-价格'},
    'xAxis'   : {'categories': ['报废机/尸体','7成新及以下','8成新','9成新','95成新','99成新', '全新']},
    'yAxis'   : {'title': {'text': '价格'}},
    
}

charts.plot(data,show='inline', options=options)
```

![](http://omg5mjb8v.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-05-10%20%E4%B8%8B%E5%8D%881.28.24.png)