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
- 每个产品都需要重新部署

## 新设计架构

### 系统结构图

![](https://raw.githubusercontent.com/JayChenFE/pic/master/tuplus/Architecure_new.png)

### 部署图

![](https://raw.githubusercontent.com/JayChenFE/pic/master/tuplus/deployment_new.png) 

## 性能优化方向

抓大放小

### 硬件的处理能力

CPU>内存>>硬盘

###  IO密集型  VS  CPU密集型 

IO密集型:

- 网络请求
- 文件读写
- 数据库读写

减少请求操作次数,异步

CPU密集型:

- 应用程序中的代码执行

  ```js
  for (var i = 0; i < Number.MAX_VALUE; i++) {
    // do something
  }
  ```

  优化算法,云计算

现有的性能问题:

- 聚集数据
  - Quartz.NET
- 批量操作
  - 官方的EF扩展库
- Excel输出
  - 封装单独的Excel操作类

## 有哪些变化?

### 数据层

统一DAL层使用的技术

#### EF ORM VS SQL?

|      | EF ORM | SQL  |
| :--: | :----: | :--: |
|  性能  |   略慢   |  快   |
| 学习成本 |   高    |  低   |
| 开发效率 |   快    |  慢   |
| 可维护性 |   高    |  低   |
| 出错率  |   低    |  高   |
| 灵活性  |   低    |  高   |

在性能要求不高的前提下使用Linq,ORM,同时保留SQL

### DbSession是什么?

`EFContextFactory` 通过`CallContext`保证db上下文唯一

`DbSession` 数据库访问层的统一入口,整个数据库访问层的抽象

作用:

- 将所有的数据仓库作为`DBSeesion` 的属性,方便使用
- `SaveChanges()` 方法避免多次连接数据库,将数据库更新权限提升到bll层更灵活

### 业务层

#### 软件工程的发展

- 瀑布流 Waterfall
- 敏捷开发 agile development
  - 迭代式增量软件开发过程 scrum
- 领域驱动设计 DDD:Domain Driven Design

#### 规约模式 Specification Pattern

```C#
public interface ICustomerRespository
 {
    Customer GetByName(string name);
    Customer GetByUserName(string userName);
    IList<Customer> GetAllRetired();
}
```

今后需要添加新的查询逻辑，结果一大堆相关代码都要修改，怎么办?

随着时间的推移，这个接口会变得越来越大，团队中你一榔头我一棒子地对这个接口进行修改，最后整个设计变得一团糟

`GetByName`和`GetByUserName` 都OK，因为语义一目了然。但是`GetAllRetired` 呢？什么是退休？

超过法定退休年龄的算退休，那么病退的是不是算在里面？

这里返回的所有Customer中，仅仅包含了已退休的男性客户，还是所有性别的客户都在里面？



规约模式就是DDD引入用来解决以上问题的一种特殊的模式。规约是一种布尔断言，它表述了给定的对象是否满足当前约定的语义。经典的规约模式实现中，规约类只有一个方法，就是`IsSatisifedBy(object);` 如下：

```C#
public class Specification
 {
    public virtual bool IsSatisifedBy(object obj)
    {
        return true;
    }
}
```

实现:

```C#
public interface ICustomerRepository
 {
    Customer GetBySpecification(Specification spec);
    IList<Customer> GetAllBySpecification(Specification spec);
}

//通过名字
public class NameSpecification : Specification
 {
    protected string name;
    public NameSpecification(string name) { this.name = name; }
    public override bool IsSatisifedBy(object obj)
    {
        return (obj as Customer).FirstName.Equals(name);
    }
}

//通过用户名
public class UserNameSpecification : NameSpecification
 {
    public UserNameSpecification(string name) : base(name) { }
    
    public override bool IsSatisifedBy(object obj)
    {
        return (obj as Customer).UserName.Equals(this.name);
    }
}

//通过年龄限制
public class RetiredSpecification : Specification
 {
    public override bool IsSatisifedBy(object obj)
    {
        return (obj as Customer).Age >= 60;
    }
}

//客户端
public class Program1
 {
    static void Main(string[] args)
    {
        ICustomerRepository cr; // = new CustomerRepository();
        Customer getByNameCustomer = cr.GetBySpecification(new NameSpecification("zhuang"));
        Customer getByUserNameCustomer = cr.GetBySpecification(new UserNameSpecification("Adam"));
        IList<Customer> getRetiredCustomers = cr.GetAllBySpecification(new RetiredSpecification());
    }
}
```

仓储接口:

```C#
public interface IRepository<TEntity>
    where TEntity : EntityObject, IAggregateRoot
{
    void Add(TEntity entity);
    TEntity GetByKey(int id);
    IEnumerable<TEntity> FindBySpecification(ISpecification spec);
    void Remove(TEntity entity);
    void Update(TEntity entity);
}
```

经典的Specification实现的问题:

时间一长，项目里充斥着各种Specification，可能其中有相当一部分都只在一个地方使用。虽然将Specification定义为类可以增加模型扩展性，但同时也会使项目变得臃肿。

引入`LINQ Expression` 

```C#
public interface ISpecification
{
    bool IsSatisfiedBy(object obj);
    Expression<Func<object, bool>> Expression { get; }
    
}

public abstract class Specification : ISpecification
{
    #region ISpecification Members
    public bool IsSatisfiedBy(object obj)
    {
        return this.Expression.Compile()(obj);
    }
    public abstract Expression<Func<object, bool>> Expression { get; }
    #endregion
}
```

- 从外部数据源获取数据时执行过滤查询，从而返回的是经过Specification过滤的结果集
- 可以在使用Specification的时候直接编写Expression，而无需创建更多的类

```C#
public abstract class Specification : ISpecification
 {
 
    public static ISpecification Eval(Expression<Func<object, bool>> expression)
    {
        return new ExpressionSpec(expression);
    }
}

internal class ExpressionSpec : Specification
{
    private Expression<Func<object, bool>> exp;
    
    public ExpressionSpec(Expression<Func<object, bool>> expression)
    {
        this.exp = expression;
    }
    
    public override Expression<Func<object, bool>> Expression
    {
        get { return this.exp; }
    }
}

class Client
{
    static void CallSpec()
    {
        ISpecification spec = Specification.Eval(o => (o as Customer).UserName.Equals("Adam"));
    }
}
```

**规约其实就是为了减小开发人员和领域专家的沟通成本。在.net中，规约的实现可以直接引用Lambda Expression，不仅实现简单，而且还能直接使用到ORM上以减小数据库查询开销。**

```C#
public IQueryable<Customer> method B(){
  IQueryable<Customer> maleCustomers = method A();
  return maleCustomers.where(x=>x.Age >= 40)
}

public IQueryable<Customer> method A(){
  return db.Customer.where(x=>x.Sex =='male');
}
```

#### 一个相对复杂的例子

#### 权限

## 如何落地

- 数据库整合
  - 名词梳理
  - ER图,表设计
- demo完善
  - 加入sql语句,存储过程的方法
  - 批量更新删除
- 业务梳理
  - 从客户端拿到参数
  - 处理业务的边界情况
  - 组织查询参数
  - 调用业务层方法
  - 过滤结果集返回
- 工具库,公用业务方法梳理
- `LinqToEF` 教程

## 前端改造

![](https://raw.githubusercontent.com/JayChenFE/pic/master/tuplus/fe_framework.png)

![](https://raw.githubusercontent.com/JayChenFE/pic/master/tuplus/fe_framework_compare.png)

​	![](https://raw.githubusercontent.com/JayChenFE/pic/master/tuplus/fe_framework_china.png)

