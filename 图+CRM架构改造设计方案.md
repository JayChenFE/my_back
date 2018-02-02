**	图+CRM架构改造设计方案**

## 现有架构的问题

### 系统结构图

![](https://raw.githubusercontent.com/JayChenFE/pic/master/tuplus/Architecure_old.png)

问题:

- PC后台和其他应用的后台不使用同一套API,代码基本不可能复用,基本职能参照业务逻辑
- 即便参照业务逻辑,因为

### 部署图

![](https://raw.githubusercontent.com/JayChenFE/pic/master/tuplus/deployment_old.png)

### 问题:

- 不同客户的网站完全是两个不相关的产品,一处有bug需要多处修改

## 新设计架构

### 系统结构图

![](https://raw.githubusercontent.com/JayChenFE/pic/master/tuplus/Architecure_new.png)

### 部署图

![](https://raw.githubusercontent.com/JayChenFE/pic/master/tuplus/deployment_new.png)  