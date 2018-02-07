# 图+CRM架构改造设计方案

## 产品定位

**小企业**可以使用的销售管理软件

## 改造方向

- 不考虑什么?
  - 极端的性能优化
  - 网站的吞吐量,高并发


- 考虑什么
  - 代码复用,提高开发效率,从根本上减少bug产生
  - 可维护性
  - 组件化,有利于个性化开发
  - 性能

## 现有架构的问题

### 系统结构图

![](https://raw.githubusercontent.com/JayChenFE/pic/master/tuplus/Architecure_old.png)

问题:

- PC后台和其他应用的后台不使用同一套API,代码基本不可能复用(`Controller层` ),基本职能参照业务逻辑
- 即便参照业务逻辑,因为后台数据层使用了不同的技术,增加了编码和单元测试的工作量,且容易产生潜在bug

### 部署图

![](https://raw.githubusercontent.com/JayChenFE/pic/master/tuplus/deployment_old.png)

### 问题:

- 不同客户的网站完全是两个不相关的产品,一处有bug需要多处修改
- 代码不能复用,容易产生一些人为的错误
- bug堆积,疲于应付需求,没有精力梳理业务

## 新设计架构

### 系统结构图

![](https://raw.githubusercontent.com/JayChenFE/pic/master/tuplus/Architecure_new.png)

### 部署图

![](https://raw.githubusercontent.com/JayChenFE/pic/master/tuplus/deployment_new.png) 

## 有哪些变化?

### 数据层

统一DAL层使用的技术

#### EF VS SQL?



