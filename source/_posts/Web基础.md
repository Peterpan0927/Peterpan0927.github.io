---
title: Web基础
date: 2017-05-26 22:39:57
tags: Web安全  前后端基础
categories: 基本知识学习
author: Peterpan
---

Web基础
<!--more-->
# Web基础

## 1.Web发展史

![24BF5CBD-1FC5-4D49-A273-4640CA1D84AE](http://omunhj2f1.bkt.clouddn.com/24BF5CBD-1FC5-4D49-A273-4640CA1D84AE.png)

各个发展时代安全问题：

![DBB2361A-A844-4014-AE83-AFBAA6756FF7](http://omunhj2f1.bkt.clouddn.com/DBB2361A-A844-4014-AE83-AFBAA6756FF7.png)

## 2.Web流程

![8009CD93-F415-4234-8377-6530045B9D53](http://omunhj2f1.bkt.clouddn.com/8009CD93-F415-4234-8377-6530045B9D53.png)

## 3.浏览器

1.首先通过域名获取web服务器IP地址

2.访问web服务器

## 4.URL协议

1.url被称作统一资源定位符，就是我们平时在网上输入的网址，可以通过收获地址的方式来进行类比。

![72994992-1FD8-47B6-B91B-3D66842C28B2](http://omunhj2f1.bkt.clouddn.com/72994992-1FD8-47B6-B91B-3D66842C28B2.png)

![378425CB-9517-47B4-A322-8E67FC5883AB](http://omunhj2f1.bkt.clouddn.com/378425CB-9517-47B4-A322-8E67FC5883AB.png)

## 5.HTTP协议

同样通过快递的方式进行类比

![8B9F21C7-27E8-4200-92D9-99948EBDE16B](http://omunhj2f1.bkt.clouddn.com/8B9F21C7-27E8-4200-92D9-99948EBDE16B.png)

![03ABEF1C-6F9A-4152-9D34-45420FAE6859](http://omunhj2f1.bkt.clouddn.com/03ABEF1C-6F9A-4152-9D34-45420FAE6859.png)

HTTP请求中的Referer:

> 告知服务器该请求的来源（浏览器自动加上）

> 统计流量：CNZZ,百度统计

> 判断来源合法性：防止盗链、防止CSRF漏洞



## 6.前端知识补充

### 1.内联框架

```html
<!--iframe 用于在网页内显示网页。-->
<!DOCTYPE html>
<html>
<body>

<iframe src="/example/html/demo_iframe.html" name="iframe_a" width="800" height="600"></iframe>

<p><a href="http://www.w3school.com.cn" target="iframe_a">W3School.com.cn</a></p>

<p><b>注释：</b>由于链接的目标匹配 iframe 的名称，所以链接会在 iframe 中打开。</p>

</body>
</html>
```

### 2.HTML DOM

![7DE16B8F-BBF0-4FF1-8E3B-F477B488EA96](http://omunhj2f1.bkt.clouddn.com/7DE16B8F-BBF0-4FF1-8E3B-F477B488EA96.png)

### 3.javaScript DOM函数

加载js的方式：HTML的script标签之间、HTML的事件属性中，如onclick、浏览器的JavaScript控制台中。

```javascript
<script>
  function changetext(id){
    id.innerHTMl = "Do you like me";
  }
</script>
<h1 onclick = "changetext(this)">请点击</h1>
```

![A5DB43A7-D8B2-4452-8F7D-3D94BBF0D593](http://omunhj2f1.bkt.clouddn.com/A5DB43A7-D8B2-4452-8F7D-3D94BBF0D593.png) 

![](http://omunhj2f1.bkt.clouddn.com/62D4BA2B-61AF-4555-924A-1AE2C2807C8C.png)

document.write写入html内容,上方样例是写入系统的时间，同样也可以写入元素。还可以在网页的源代码上进行自定义的修改。

以上使用JavaScript访问和操作HTML就是JavaScript DOM的操作

DOM的本质就是连接Web页面和编程语言，JavaScript + DOM访问和操作HTML文档的标准方法。



### 4.JavaScript BOM函数

获取浏览器信息，操作浏览器行为，连接浏览器和编程语言

> 警告弹窗 alert()
>
> 确认弹窗 confirm()
>
> 提示弹窗 prompt()

![D549F44E-070D-4884-85DA-1BCFF555F078](http://omunhj2f1.bkt.clouddn.com/D549F44E-070D-4884-85DA-1BCFF555F078.png)

TIPS:常用于简单的调试和信息展示，如XSS漏洞的测试。 

具体的一些操作行为；

window.location:获取／控制用户页面URL

window.location = "http://www.baidu.com"是直接进行页面的跳转

window.screen:获取浏览器屏幕信息

window.navigator:获取访问者浏览信息

window.open/close:操作浏览器窗口



## 7.服务端环境

### 1.Web服务端概述:web服务端包括数据服务和web服务

静态页面时期：![6517E706-913F-439B-AA1E-1FFC8FD83FDF](http://omunhj2f1.bkt.clouddn.com/6517E706-913F-439B-AA1E-1FFC8FD83FDF.png)

动态页面时期：php文件通过语言解释器调用数据库中的数据，数据库返回相应数据，通过语言解释器传递给服务器相应的html文件，使用户端得以访问。![C6E24980-02F9-434B-BD52-0714115FBB83](http://omunhj2f1.bkt.clouddn.com/C6E24980-02F9-434B-BD52-0714115FBB83.png)

目前流行的架构分析：

| 操作系统           | web服务  | 解释执行环境    | 数据库服务      | WEB服务端 |
| -------------- | ------ | --------- | ---------- | ------ |
| Windows server | IIS    | ASP(.NET) | SQL Server | .NET   |
| Linux          | Apache | PHP       | MySQL      | LAMP   |
| UNIX/Windows   | Tomcat | JSP       | Oracle     | J2EE   |

Apache服务：



![85235B63-733C-46FD-90D4-F6C81CB72E9F](http://omunhj2f1.bkt.clouddn.com/85235B63-733C-46FD-90D4-F6C81CB72E9F.png)

其中DNS服务器主要是针对公网访问，如果只是本机访问可以在HOSTS文件中自定义域名。

![13A5F48C-4B16-4F56-98E4-BB4C97BD1FDB](http://omunhj2f1.bkt.clouddn.com/13A5F48C-4B16-4F56-98E4-BB4C97BD1FDB.png)

### 2.SQL概述(结构化查询语言)

与网络商城对比可以更加形象的理解

![](http://omunhj2f1.bkt.clouddn.com/DF076950-BC52-4C06-9A8A-983058605E18.png)

数据库操作：

| 创建数据库 | CREATE DATABASE websecurity; |
| ----- | ---------------------------- |
| 查看数据库 | SHOW databases;              |
| 切换数据库 | USE websecurity;             |
| 删除数据库 | DROP DATABASE websecurity;   |

TIPS:SQL语句对大小写不敏感，分号作为语句的结束，程序中会自动补充

表的操作：

| 创建表  | CREATE  TABLE teacher( id int(4) not null primary key auto_increment, name char(20) not null, sex char(10) not null, addr char(20) not null ); |
| ---- | ---------------------------------------- |
| 删除表  | DROP TABLE teacher; DELETE FROM teacher name="xxx"; |
| 插入表  | INSERT INTO teacher (name, sex, addr) VALUES ('pzp', 'male', 'wuhan'); |
| 查询表  | select * from teacher;                   |
| 更新表  | UPDATE teacher SET name = 'neo' where addr="wuhan" |

SQL语法讲解：

1.where语句

```mysql
#进行需要信息的查找
select 你要的信息 from 数据表（或多个） where 满足的条件（条件判断）
select name from teacher where addr="wuhan" and sex="male" 
```

2.order by语句

```mysql
select 你要的信息 from 数据表（或多个） order by 字段 ASC/DESC
select * from teacher order by name #对名字进行升序排序，默认是ASC,name也可以用数字替代，代表是第几类
```

3.UNION语句

```mysql
#合并两张表,不会显示重复的数据，如果要显示重复的数据，要用union all
select 你要的信息 from 数据表1 union select 你要的信息 from 数据表2
select name from teacher union select name from student
```

4.导入sql文件

```mysql
#注意，因为mysql会自动注释掉斜杠，所以在路径中要多加一个‘／’
source 文件路径 sql
```

5.常见内置函数[参考](http://www.cnblogs.com/noway-neway/p/5211401.html)

```mysql
#数学函数
abs(x),sqrt(x),exp(x)...
#字符串函数
INSERT(str,pos,len,newstr)
/*返回str,其起始于pos，长度为len的子串被newstr取代。
1. 若pos不在str范围内，则返回原字符串str
2. 若str中从pos开始的子串不足len,则将从pos开始的剩余字符用newstr取代
3. 计算pos时从1开始，若pos=3,则从第3个字符开始替换
*/...
#日期和时间函数
UTC_DATE():返回当前世界标准时间
EXTRACT(unit FROM date)：提取日期时间中的要素
e.g-> SELECT EXTRACT(YEAR FROM '2009-07-02'); ##2009
#条件判断函数
IF(expr1,expr2,expr3)
#如果expr1不为0或者NULL,则返回expr2的值，否则返回expr3的值
IFNULL(expr1,expr2)
#如果expr1不为NULL,返回expr1,否则返回expr2
CASE value WHEN [compare_value] THEN result [WHEN [compare_value] THEN result ...] [ELSE result] END
#当compare_value=value时返回result
CASE WHEN [condition] THEN result [WHEN [condition] THEN result ...] [ELSE result] END
#当condition为TRUE时返回result
#系统信息函数
DATABASE()，SCHEMA()：显示当前使用的数据库
#格式或类型转化函数
FORMAT(X,D[,locale])：将数字X转化成'#,###,###.##'格式，D为保留的小数位数
CONV(N,from_base,to_base)：改变数字N的进制，返回值为该进制下的数字构成的字符串
```

## 3.php概述（超文本预处理器）

### php基本语法的讲解

![](http://omunhj2f1.bkt.clouddn.com/3831EE56-8DD6-4407-9581-28479A56A625.png)

串接点的作用是连接不同的字符串

TIPS：变量大小写代表的是不同的变量

![440C424F-04CE-43EE-9AA2-11A58879F254](http://omunhj2f1.bkt.clouddn.com/440C424F-04CE-43EE-9AA2-11A58879F254.png)

### php实例（一），返回表单中的信息：

```php
TIPS：当http请求类型为GET的时候，后端变量就要用$_GET，如果请求类型是POST的话，后端变量就要用$_POST，当然还可以用$_REQUEST,可以接受两种传值。
```

![E514CDDC-3EC9-43EC-B02D-C8C36CEEF159](http://omunhj2f1.bkt.clouddn.com/E514CDDC-3EC9-43EC-B02D-C8C36CEEF159.png)

### php实例（二），文件上传：

> 在黄色部分判断文件是否存在
>
> 红色部分打印文件的信息
>
> 绿色部分将文件保存在相应的目录下



![upload_file.php](http://omunhj2f1.bkt.clouddn.com/24BF7A3C-7A6C-4E8D-810E-B69B88379C14.png)

php中常见的系统变量：

| $_GLOBALS[] | 储存当前脚本中的所有全局变量，其KEY为变量名，VALUE为变量值        |
| ----------- | ---------------------------------------- |
| $SERVER[]   | 当前WEB服务器变量数组                             |
| $GET[]      | 存储以GET方法提交表单中的数据                         |
| $_POST[]    | 存储以POST方法提交表单中的数据                        |
| $_COOKIE[]  | 取得或设置用户浏览器Cookies中存储的变量数组                |
| $_FILES[]   | 存储上传文件提交到当前脚本的数据                         |
| $_ENV[]     | 存储当前WEB环境变量                              |
| $_REQUEST[] | 存储提交表单中所有请求数组，其中包括$_GET、$_POST、$_COOKIE和$_SESSION中的所有内容 |
| $_SESSION[] | 存储当前脚本的会话变量数组                            |

打印服务器相关的信息；

```php
<?php
  echo "服务器名称：".$_SERVER['SERVER_NAME']."<br>";
  echo "网站根目录：".$_SERVER['DOCUMENT_ROOT']."<br>";
  echo "当前网页相对路径：".$_SERVER['PHP_SELF']."<br>";
  echo "当前网页绝对路径：".$_SERVER['SCRIPT_FILENAME']."<br>";
  echo "服务器环境变量：".$_SERVER['PATH']."<br>";
include 'upload_file.php';
?>
```

TIPS:include/require:包含文件

include:警告，脚本继续

require:错误，停止脚本

### 使用php对数据库进行操作

![](http://omunhj2f1.bkt.clouddn.com/7B4413D8-D8AA-4204-97E2-818C0E91A648.png)