---
title: 软件系统分析与设计（九）
urlname: swsad-hw9
tags:
  - SWSAD
  - HomeWork
copyright: true
category: SWSAD
date: 2019-05-04 20:57:26
---

> 系统分析与设计 作业九

<!-- more --> 

## 1、使用类图，分别对 Asg_RH 文档中 Make Reservation 用例以及 Payment 用例开展领域建模。然后，根据上述模型，给出建议的数据表以及主要字段，特别是主键和外键

> - 注意事项：
>   - 对象必须是名词、特别是技术名词、报表、描述类的处理；
>   - 关联必须有多重性、部分有名称与导航方向
>   - 属性要注意计算字段
> - 数据建模，为了简化描述仅需要给出表清单，例如：
>   - Hotel（ID/Key，Name，LoctionID/Fkey，Address…..）
>

### MakeReservation

![1557063199381](https://blog.janking.cn/post/swsad-hw9/1557063199381.png)

### Payment

![1557065284871](https://blog.janking.cn/post/swsad-hw9/1557065284871.png)

- User(ID,Name,Email,Phone,Gender)
- Reservation(ID/Key,UserID/Fkey,HotelID/Fkey,Roomcount,StartTime,EndTime,TotalPrice)
- Hotel(ID/Key,Name,Address,City)
- Room(ID/Key,HotelID/Fkey,Name,Tpte,Price)
- Payment(ID/Key,Reservation/Fkey,Money)
- ShopBasket(UserID/Fkey)
- CreditCard(ID/Key,Type)

## 2、使用 UML State Model，对每个订单对象生命周期建模

> - 建模对象： 参考 Asg_RH 文档， 对 Reservation/Order 对象建模。
> - 建模要求： 参考练习不能提供足够信息帮助你对订单对象建模，请参考现在 定旅馆 的旅游网站，尽可能分析围绕订单发生的各种情况，直到订单通过销售事件（柜台销售）结束订单。
>

![1557066434457](https://blog.janking.cn/post/swsad-hw9/1557066434457.png)