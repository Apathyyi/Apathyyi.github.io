---
layout: post
title: list转json并只转换必要项，数据库datetime和date类型如何正确在jsp页面显示.
category: 
keywords:json,fastjson,date.
excerpt: 学海无涯
---

今天写crm系统联系人界面时,需要异步得到客户的信息。后端需要将数据库查询到的数据转换为json传给前端页面，但是再转换时出现了stackoverflow.如何在正确显示日期.

## list转json
我使用的时fastjson,阿里爸爸开发的一款专门用于Java开发的包，可以方便的实现json对象与JavaBean对象的转换，实现JavaBean对象与json字符串的转换，实现json对象与json字符串的转换
使用之前需要先导入jar包,下载地址：https://github.com/alibaba/fastjson.
接下来开始使用fsatjson,首先我从数据库取出的是一个集合.
```java
List<Customer> list = customerService.findAll();
```
如果不需要去除不要项，那么可以直接转换为json传给前端界面
```
ServletActionContext.getResponse().getWriter().write(JSONObject.toJSONString(list,filter));
```
由于customer类里有linkman的set集合，同时linkman类有customer的一个对象并且两个类我都重写了tostrin()方法，所以在转换的时候会出现无限循环.
因为customer在tostring时需要linkman调用tostring方法而linkman也需要cutomer调用tostring方法，这样就出现了循环调用,最终导致stackoverflow,只需要在任意一方去掉对方的tostring中属性即可
例如在customer的tostring()中不将linkman转换.
如果你本来前端界面就不需要这么多冗余的数据，那么就可以只讲你需要的字段转为json传递过去，我们只需要这样：
```
SimplePropertyPreFilter filter = new SimplePropertyPreFilter(Customer.class, "cust_id","cust_name");
JSONObject.toJSONString(list,filter);
```
这样就可以直接获取到我们想要的字段了.
##关于jsp页面日期显示的问题
由于我的目的为了学习这些方法，所以我在数据库存的两个字段分别是datetime(带有时分秒)和date类型，再按照时间进行查询的时候需要查询完之后时间仍然回显.
第一种：如果不需要id的话可以直接struts的标签或者是fmt标签
```
<s:date name="visit_time" format="yyyy-mm-dd"/>              
<fmt:formatDate value="${visit_time}"  pattern="yyyy-mm-dd"/>	
```
由于我需要使用日期控件要求标签有一个id来加上控件,所以只能在input里面嵌套一个s标签.
```
<input width="180px" maxlength="50" name="visit_time" id="visit_time" value='<s:date name="visit_time" format="yyyy-mm-dd"/>'/>
```
大概效果如下:</br>
![](https://github.com/Apathyyi/Apathyyi.github.io/blob/master/image/date1.png)
</br>
![](https://github.com/Apathyyi/Apathyyi.github.io/blob/master/image/date2.png)
</br>
有什么问题可以提出,希望大家共同学习.
