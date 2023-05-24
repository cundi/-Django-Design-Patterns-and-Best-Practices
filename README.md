-------------------------  
## 简介

## 版权协议

除注明外，所有文章均采用 [Creative Commons BY-NC-ND 3.0（自由转载-保持署名-非商用-非衍生）](http://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh) 协议发布。

这意味着你可以在非商业的前提下免费转载，但同时你必须：

* 保持文章原文，不作修改。
* 明确署名，即至少注明 `作者：cundi` 字样以及文章的原始链接。


# Django-Design-Patterns-and-Best-Practices
中文名：《Django 设计模式与最佳实践》  
英文原版：https://www.packtpub.com/web-development/django-design-patterns-and-best-practices  
作者：Arun Ravindran  
出版日期：March 2015  
特色：Easily build maintainable websites with powerful and relevant Django design patterns  
级别：Mastering   
页数：Paperback 222 pages  


第三章预览
***********
第三章 模型
-----------  

本章，我们会讨论以下话题：  


* 模型的重要性  
* 类图表  
* 模型的结构模式  
* 模型的行为模式  
* 迁移  

## M大于V与C
在Django中，模型就是类，该类提供了一种处理数据库的面向对象方法。通常，每个类都引用一个数据库表，每个属性都引用一个数据库列。你可以使用一个自动生成的API来查询这些表。  

模型是很多其他组件的基础。只要你有一个模型，你可以很快地得到模型admin，模型表单，以及所有类型的通用视图。每种情况下，都需要你编写一两行代码，这样可以让它看上去没有太多魔法。  

模型也被用在更多的超出你期望的地方。这是因为Django可以以多种方式运行。Django的一些切入点如下：  

* 常见的web请求-响应流程  
* Django的交互式命令行  
* 管理命令  
* 测试脚本  
* 异步任务队列，比如Celery  

几乎所有的情况中，模型模块都需要导入（作为django.setup()的一部分）。因此，最好保证模型远离任何不必要的依赖，或者导入任何的其他Django组件，比如视图。  

简而言之，恰当地设计模型是件十分重要的事情。现在，让我们从SuperBook模型设计开始。  

>#### 注释
**自带午餐便当**  
*作者注释：SuperBook项目的进度会以这样的一个盒子来表现。你可以跳过这个盒子，但是在web应用项目中的情况下，你缺少的是领悟，经验 。  

>史蒂夫和客户的第一周——超级英雄情报监控（简称为S.H.I.M）。简单来说，这是一个大杂烩。办公室是非常未来化的，但是不论做什么事情都需要上百个审核和签字。  

>作为Django开发者的领队，史蒂夫已经配置好了中型的运行超过两天的4台虚拟机。第二天的一个早晨，机器自己不翼而飞了。一个附近的清洁机器人说，机器被法务部们给带走了，他们要对未经审核的软件安装做出处理。  

>然而，CTO哈特给予史蒂夫了极大的帮助。他要求机器在一个小时之内完好无损地给还回去。他还对SuperBook项目做出了提前审核以避免将来可能出现的任何阻碍。  

>那个下午的稍晚些时候，史蒂夫给他带了一个午餐便当。身着一件米色外套和浅蓝色牛仔裤的哈特如约而至。尽管高出周围人许多，有着清爽面庞的他依旧那么帅气，那么平易近人。他问史蒂夫如果他之前是否尝试过构建一个60年代的超级英雄数据库。  

>”嗯，对的，是哨兵项目么？“，史蒂夫说道。”是我设计的。数据库看上去被设计成了一个条目-属性-值形式的模式，有些地方我考虑用反模式。可能，这些天他们有一些超级英雄属性的小想法。哈特几乎等不到听完最后一句，他压低嗓门道：“没错。是我的错。另外，他们只给了我两天来设计整个架构。他们这是在要我老命啊！”  

>听了这些，史蒂夫的嘴巴张的大大的，三明治也卡在了嘴里。哈特微笑着道：“当然了，我还没有尽全力来做这件事。只要它成长为100万美元的单子，我们就可以多花点时间在这该死的数据库上了。SuperBook用它就能分分钟完事的，小史你说呢？”  

>史蒂夫微微点头称是。他从来没有想过在这么样的地方将会有上百万的超级英雄出现。  


## 模型搜寻
这是我们头一次见识到SuperBook中模型。我们只表示了基本模型，以及类图表中的表单的基本关系，这也是早期尝试中所特有的情况：   

![2015-05-28 14 53 47](https://cloud.githubusercontent.com/assets/10941075/7854171/6549c63c-0549-11e5-8dc9-3550ff9edcdb.png)


让我们暂且忘掉模型，来谈谈我们正在构建的对象的术语。每个用户都有一个账户。用户可以写多个回复或者多篇文章。**Like**同时关联到了一个独立用户/文章组合。  

建议你为自己的模型画一个这样类图表。这一步的某些属性缺失了，不过你可以在之后对它们进行详细补充。只要整个项目用图表表现出来，便可以轻松地分离app了。  

下面是创建这个表现的一些提示：  


* 盒子表示条目，它将成为模型。  
* 名词通常作为条目的终止。  
* 箭头是双向的，它代表了Django中的三种关系类型其中的一种：一对一，一对多（通过外键实现），和多对多。  
* 字段表明在模型中根据**条目-关系模型（ER-modle）**定义了一对多关系。换句来说，星号就是声明外键的地方。  

类图表可以映射到下面的Django代码中（分布于多个应用之中）：  

```python
class Profile(models.Model):
    user = models.OnToOneField(User)

class Post(models.Model):
    posted_by = models.ForeignKey(User)

class Comment(models.Model):
    commented_by = models.ForeignKey(User)
    for_post = models.ForeignKey(Post)

class Like(models.Model):
    liked_by = models.ForeignKey(User)
    post = models.ForeignKey(Post)
```

后面，我们不会直接地引用**User**，而是使用更常见的**settings.AUTH_USER_MODEL**来。  


### 把model.py分到多个文件中去
就像多数的Django组件那样，一个大的model.py文件可以在一个包内分割为多个文件。**package**通过一个目录来实现，它包含多个文件，目录中的一个文件必须是一个称为`__init__.py`特殊文件。  

所有可以在包级别中暴露的定义都必须在`__init__.py`里使用全局变量域定义。例如，如果我们分割model.py到独立的类，models子文件夹中的对应文件，比如，postable.py，post.py和comment.py, 之后`__init__.py`包会像这样：  

```python
    from postable import Postable
    from post import Post
    from commnet import Comment
```  

现在你可以像之前那样导入models.Post了。  

在`__init__.py`包中的任何其他代码都会在包运行时被导入。因此，它是一个任意级别包初始化代码的理想之地。  

## 结构模式
本节包含多个帮助你设计和构建模型的设计模式。  

### 模式-规范化模型
**问题**：通过设计，模型实例的重复数据引起数据不一致。  

**解决方法**：通过规范化，分解模型到更小的模型。使用这些模型之间的逻辑关系来连接他们。  

### 问题细节
想象一下，如果某人用下面的方法设计Post表（省略部分列）：  


| 超级英雄的名字        | 消息          | 发布时间  |
| ------------- |:-------------:| -----:|
| Captain Temper      | 消息已经发布过了？ | 2012/07/07/07:15 |
| Professor English      | 应该用“Is”而不是“Has"      |   2012/07/07/07:17 |
| Captain Temper | 消息已经发布过了？      |  2012/07/07/07:18 |
| Capt. Temper | 消息已经发布过了？      |  2012/07/07/07:19 |

  
我希望你注意到了在最后一行的超级英雄名字和之前不一致（船长一如既往的缺乏耐心）。  

如果我们看看第一列，我们也不确定哪一个拼写是正确的——`Captain Temper或者Capt.Temper`。这就是我们要通过规范化消除的一种数据冗余。  

### 详解
在我们看下完整的规范方案，让我们用Django模型的上下文来个关于数据库规范化的简要说明。  

### 规范化的三个步骤
规范化有助于你更有效地的存储数据库。只要模型完全地的规范化处理，他们就不会有冗余的数据，每个模型应该只包含逻辑上关联到自身的数据。  

这里给出一个简单的例子，如果我们规范化了Post表，我们就可以不模棱两可地引用发布消息的超级英雄，然后我们需要用一个独立的表来隔离用户细节。默认，Django已经创建了用户表。因此，你只需要在第一列中引用发布消息的用户的ID，一如下表所示：  

|用户ID|消息    |发布时间   |
|-----|:------:|---------:|
| 12      | 消息已经发布过了？ | 2012/07/07/07:15 |
| 8      | 应该用“Is”而不是“Has"      |   2012/07/07/07:17 |
| 12 | 消息已经发布过了？      |  2012/07/07/07:18 |
| 12 | 消息已经发布过了？      |  2012/07/07/07:19 |
  

现在，不仅仅相同用户发布三条消息的清楚在列，而且我们可以通过查询用户表找到用户的正确的名字。  

通常来说，你会按照模型的完全规规范化表来设计模型，也会因为性能原因而有选择性地非规范化设计。在数据库中，**Normal Forms**是一组可以被应用于表，确保表被规范化的指南。一般我们建立第一，第二，第三规范表，尽管他们可以递增至第五规范表。  

这接下来的例子中，我们规范化一个表，创建对应的Django模型。想象下有个名字称做“Sightings”的表格，它列出了某人第一次发现超级英雄使用能力或者特异功能。每个条目都提到了已知的原始身份，超能力，和第一次发现的地点，包括维度和经度。  

| 名字       | 原始信息     | 能力 |第一次使用记录（维度，经度，国家，时间） |
| ----------| :-----------:|:----:|----------------------------------:|
|Blitz      | Alien        |Freeze Flight|+40.75, -73.99; USA; 2014/07/03 23:12
|Hexa       |Scientist      |Telekinesis Flight|+35.68, +139.73; Japan; 2010/02/17 20:15
|Traveller  |Billonaire     |Time travel    |+43.62, +1.45, France; 2010/11/10 08:20

上面的地理数据提取自http://www.golombek.com/locations.html.  

### 第一范式表（1NF）
确认一下的第一张范式表格，这张表必须含有：  

    多个没有属性（cell）的值
    一个主键作为单独一列或者一组列（合成键）

让我们试着把表格转换为一个数据库表。明显地，我们的`Power`列破坏了第一个规则。  

更新过的表满足第一范式表。主键（用一个`*`标记）是`Name`和`Power`的合并，对于每一排它都应该是唯一的。  

|Name*|Origin|Power*|Latitude   |Longtitude |Country|Time       |
|-----|:----:|:----:|:---------:|:---------:|:-----:|:---------:|
|Blitz |Alien |Freeze|+40.75170  |-73.99420|USA|2014/07/03 23:12|
|Blitz|Alien|Flight|+40.75170|-73.99420|USA|2013/03/12 11:30|
|Hexa|Scientist|Telekinesis|+35.68330|+139.73330|Japan|2010/02/17 20:15|
|Hexa|Scientist|Filght|+35.68330|+139.73330|Japan|2010/02/19 20:30|
|Traveller|Billionaire|Time tavel|+43.61670|+1.45000|France|2010/11/10 08:20|



### 第二范式表
第二范式表必须满足所有第一范式表的条件。此外，它必须满足所有非主键列都必须依赖于整个主键的条件。  

我们注意到在前面的表中`Origin`只依赖于超级英雄，即，`Name`。不论我们谈论的是哪一个`Power`。因此，`Origin`不是完全地依赖于合成组件-`Name`和`Power`。  

这里，让我们只取出原始信息到一个独立的，称做`Origins`的表：  

|Name*|Origin|
|-----|-----:|
|Blitz|Alien|
|Hexa|Scientist|
|Traveller|Billionaire|
  

现在`Sightings`表更新为兼容第二范式表，它大概是这个样子：  

|Name*|Power*|Latitude   |Longtitude |Country|Time       |
|-----|:----:|:---------:|:---------:|:-----:|:---------:|
|Blitz |Freeze|+40.75170  |-73.99420|USA|2014/07/03 23:12|
|Blitz||Flight|+40.75170|-73.99420|USA|2013/03/12 11:30|
|Hexa|Telekinesis|+35.68330|+139.73330|Japan|2010/02/17 20:15|
|Hexa|Filght|+35.68330|+139.73330|Japan|2010/02/19 20:30|
|Traveller|Time tavel|+43.61670|+1.45000|France|2010/11/10 08:20|
  

### 第三范式表
在第三范式表中，比表格必须满足第二范式表，而且应该额外满足所有的非主键列都直接依赖整个主键，而且这些非主键列都是互相独立的这个条件。  

考虑下`Country`类。给出`维度`和`经度`，你可以轻松地得出`Country`列。即使观测到超级英雄的地方依赖于`Name-Power`合成键，但只是间接地依赖他们。  

因此，我们把详细地址分离到一个独立的国家表格中：  

|Location ID|Latitude*|Longtitude*|Country|
|-----------|:-------:|:---------:|------:|
1|+40.75170|-73.99420|USA|
2|+35.68330|+139.73330|Japan|
3|+43.61670|+1.45000|France|
  

现在`Sightings`表格的第三范式表大抵如此：  

|User ID*|Power*|Location ID|Time|
---------|:----:|:---------:|---:|
2|Freeze|1|2014/0703 23:12
2|Flight|1|2013/03/12 11:30
4|Telekinesis|2|2010/02/17 20:15
4|Flight|2|2010/02/19 20:30
7|Time tavel|3|2010/11/10 08:20
  

如之前所做的那样，我们用对应的`User ID`替换了超级英雄的名字，这个用户ID用来引用用户表格。

### Django模型
现在我们可以看看这些规范化的表格可以用来表现Django模型。Django中并不直接支持合成键。这里用到的解决方案是应用代理键，以及在`Meta`类中指定`unique_together`属性：  

```python
    class Origin(models.Model):
        superhero = models.ForeignKey(settings.AUTH_USER_MODEL)
        origin = models.CharField(max_length=100)

    class Location(models.Model):
        latitude = models.FloatField()
        longtitude = models.FloatField()
        country = models.CharField(max_length=100)

        class Meta:
            unique_together = ("latitude", "longtitude")

    class Sighting(models.Model):
        superhero = models.ForeignKey(settings.AUTH_USER_MODEL)
        power = models.CharField(max_length=100)
        location = models.ForeignKey(Location)
        sighted_on = models.DateTimeField()

        class Meta:
            unique_together = ("superhero", "power")
```  


### 性能和非规范化
规范化可能对性能有不利的影响。随着模型的增长，需要应答查询的连接数也随之增加。例如，要在美国发现具有冷冻能力的超级英雄的数量，你需要连接四个表格。先前的内容规范后，任何信息都可以通过查询一个单独的表格被找到。  

你应该设计模式以保持数据规范化。这可以维持数据的完整。然而，如果你面临扩展性问题，你可以有选择性地从这些模型取得数据以生成非规范化的数据。  
  
>##### 提示
**最佳实践**
*因设计而规范，又因优化而非规范*  

>例如，在一个确定的国家中计算观测次数是非常普通的，然后将观测次数作为一个附加的字段到`Location`模型。现在，你可以使用Django ORM 继承其他的查询，而不是一个缓存的值。  

>然而，你需要在每次添加或者移除观测时更新这个计数。你需要添加该计算到*Singhting*的`save`方法，添加一个信号处理器，甚至使用一个异步任务去计算。  

>如果你有一个跨越多个表的负责查询，比如国家的超能力计算，你需要创建一个独立的非规范表格。就像前面那样，我们需要在每一次规范化模型中的数据改变时更新这个非规范的表格。  

>令人惊讶的是非规范化在大型的网站中是非常普遍的，因为它是数度和存储空间两者之间的折衷。今天的存储空间已经比较便宜了，然而速度也是用户体验中至关重要的一环。因此，如果你的查询耗时过于久的话，那么就需要考虑非规范化了。  

## 我们应该一直使用规范化吗？
过多的规范化是是件不必要的事。有时候，它可以引入一个非必需的能够重复更新和查询的表格。  

例如，你的`User`模型或许有好多个家庭地址的字段，你可以规范这些字段到一个`Address`模型中。可是，多数情况下，把一个额外的表引进数据库是没有必要的。  

与其针对大多数的非规范化设计，不如在代码重构之前仔细地衡量每个非规范化的机会，对性能和速度上做出一个折衷的选择。

  
## 模式-模型mixins
**问题**：明显地模型含有重复的相同字段/或者方法，违反了DRY原则。  

**方案**：提取公共字段和方法到各种不同的可重复使用的模型mixins中。  

## 问题细节
设计模型之时，你或许某些个公共属性或者行为跨类共享。烈日，`Post`和`Comment`模型需要一直跟踪自己的`created`日期和`modified`日期。手动地复制-粘贴字段和它们所关联的方法并不符合DRY原则。  

由于Django的模型是类，像合成以及继承这样的面向对象方法都是可以选择的解决方案。然而，合成（具有包含一个共享类实例的属性）需要一个额外的间接地访问字段的标准。  

继承是有技巧的。我们使用一个`Post`和`Comment`的公共基类。然而，在Django中有三种类型的继承：**concrete（具体）**, **abstract（抽象）**, 和**proxy（代理）**。  

具体继承的运行是源于基类，就像你在Python类中通常用到的那样。不过，在Django中，这个基类将被映射到一个独立的表中。每次你访问基本字段时，都需要一个准确的连接。这会带来非常糟糕的性能问题。  

代理继承只能添加新的行为到父类。你不能够添加新字段。因此，这种情况下它也不大好用。  

最后，我们只有托付于抽象继承了。

## 详解  
抽象基类是用于模型之间共享数据和行为的游戏简洁方案。当你定义一个基类时，它在数据中没有创建任何与之对象的表。反而，这些字段是在派生的非基类中创建的。  

访问抽象基类字段不要`JOIN`语句。有支配的字段结构表也是不解自明的。对于这些优势，大多数的Django项目都使用抽象基类实现公共字段或者方法。  

使用抽象模型是局限的：  
    
    它们不能够拥有外键或者来自其他模型的多対多字段。
    它们不能够被实例化或者保存。
    它们不能够直接用在查询中，因为它没有管理器。  

下面是post和comment类如何使用一个抽象基类初始设计的：  

```python
    class Postable(models.Model):
        created = models.DateTimeField(auto_now_add=True)
        modified = modified.DateTimeField(auto_now=True)
        message = models.TextField(max_length=500)

        class Meta:
            abstract = True


    class Post(Postable):
        ...


    class Comment(Postable):
        ...
```  

要将一个模型转换到抽象基类，你需要在它的内部`Meta`类中写上`abstract = True`。这里的`Postable`是一个抽象基类。可是，它不是那么的可复用。  

实际上，如果有一个类含有`created`和`modified`字段，我们在后面就可以在附近的任何需要时间戳的模型中重复使用这个时间戳功能。  

### 模型mixins
模型mixins是一个可以把抽象基类当作父类来添加的模型。不像其他的语法，比如Java那样，Python支持多种继承。因此，你可以列出一个模型的任意数量的父类。  

Mixins应该是互相垂直的而且易于组合的。把一个mixin放进基类的列表，这些mixin应该可以正常运行。这样看来，它们在行为上更类似于合成而非继承。  

较小的mixin的会更好。不论何时一个mixin变得臃肿，而且又违反了独立响应原则，就要考虑把它重构到一个更小的类。就让一个mixin一次做好一件吧。  

在前面的例子中，模型mixin用于更新`created`和`modified`的时间可以轻松地分解出来，一如下面代码所示：  

```python
    class TimeStampedModel(models.Model):
        created = modified.TimeStampModel(auto_now_add=True)
        modified = modified.DateTimeField(auto_now=True)

        class Meta:
            abstract = True


    class Postable(TimeStampedModel):
        message = models.TextField(max_length=500)
        ...

        class Meta:
            abstract = True


    class Post(Postable):
        ...


    class Comment(Postable):
        ...
```  
  
我们现在有两个超类了。不过，功能之间都完全地独立。mixin可以分离到自己的模块之内，并且另外的上下文中被复用。  
   
## 模式-用户账户
**问题**：每一个网站都存储一组不同的用户账户细节。然而，Django的内建User模型旨在针对认证细节。  

**方案**：用一个一对一关系的用户模型，创建一个用户账户类。  

## 问题细节
Django提供一个开箱即用的相当不错的User模型。你可以在创建超级用户或者登录amdin接口的时候用到它。它含有少量的基本字段，比如全名，用户名，和电子邮件。  

然而，大多数的现实世界项目都保留了很多关于用户的信息，比如他们的地址，喜欢的电影，或者它们的超能力。打Django1.5开始，默认的User模型就可以被扩展或者替换掉。不过，官方文档极力推荐只存储认证数据，即便是在定制的用户模型中也是如此（毕竟，用户模型也是所属于`auth`这个app的）。  

某些项目是需要多种类型的用户的。例如，SuperBook可以被超级英雄和非超级英雄所使用。这里或许会有一些公共字段，以及基于用户类型的不同字段。  

## 详解
官方推荐解决方案是创建一个用户账户模型。它应该和用户模型有一个一对一的关系。其余的全部用户信息都存储于该模型： 

```python
    class Profile(models.Model):
        user = models.OnToOneField(settings.AUTH_USER_MODEL, primary_key=True)
```  

这里推荐你明确的将`primary_key`赋值为`True`以阻止类似PostgreSQL这样的数据库后端中的并发问题。剩下的模型可以包含其他的任何用户详情，比如生日，喜好色彩，等等。  

设计账户模型之时，建议所有的账户详情字段都必须是非空的，或者含有一个默认值。凭直觉我们就知道用户在注册时是不可能填写完所有的账户细节的。此外，我们也要确保创建账户实例时，信号处理器没有传递任何初始参数。  

### 信号
理论上，每一次用户模型的创建都必须把对应的用户账户实例创建好。这个操作通常利用信号来完成。例如，我们可以使用下面的信号处理器侦听用户模型的`post_save`信号：  

```python
    # signals.py
    from django.db.models.signals import post_save
    from django.dispatch import receiver
    from django.conf import settings
    from . import models


    @receiver(post_save, sender=settings.AUTH_USER_MODEL)
    def create_profile_handler(sender, instance, created, **kwargs):
        if not created:
            return
        # Create the profile object, only if it is newly created
        profile = models.Profile(user=instance)
        profile.save()
```  

注意账户模型除了用户实例之外没有传递额外的参数。  

之前没有指定初始信号代码的地方。通常，它们在`models.py`中（这是不可靠的）导入或者执行。不过，随着Django 1.7的app载入重构，应用初始化代码位置的问题也很好的解决了。  

首先，为你的应用创建一个`__init__.py`包以引用应用的`ProfileConfig`:  

    default_app_config = "profile.apps.ProfileConfig"

接下来是`app.py`中，子类`ProfileConfig`的方法，可于`ready`方法中配置信号：  

```python
    # app.py
    from django.apps import AppConfig


    class ProfileConfig(AppConfig):
        name = "profiles"
        verbose_name = "User Profiles"


        def ready(self):
            from . import signals
```  

随着信号的配置，对所有的用户来说，访问`user.profile`应该都返回一个`Profile`对象，即使是最新创建的用户也是如此。  

### Admin
现在，用户的详情为存在admin内的两个不同地方：普通用户admin页面中的认证细节，在一个独立账户admin页面中的相同用户的补充账户详情。但是这样做非常麻烦。  

为了操作方便，账户admin可以通过定义一个自定义的`UserAdmin`嵌入到默认的用户admin中：  

```python
    # admin.py
    from django.contrib import admin
    from .models import Profile
    from django.contrib.auth.models import User


    class UserProfileInline(admin.StackedInline):
        model = Profile


    class UserAdmin(admin.UserAdmin):
        inlines = [UserProfileInline]


    admin.site.unregister(User)
    admin.site.register(User, UserAdmin)
```  
  
### 多账户类型
假设在应用中你需要几种类型的用户账户。这里需要有一个字段去跟踪用户使用的是哪一种账户类型。账户数据本身需要存储在独立的模型中，或者存储在一个统一的模型中。  

建议使用聚合账户的方法，因为它让改变账户类型而不丢失账户细节，并具有灵活性，减小复杂度。在这个房中中，账户模型包含一个所有账户类型的字段超集。  

例如，SuperBook会需要一个`SuperHero`类型账户，和一个`Ordinary`（非超集英雄）账户。它可以用一个独立的统一账户模型实现：  

```python
    class BaseProfile(models.Model):
        USER_TYPES = (
            (0, 'Ordinary'),
            (1, 'SuperHero'),
        )
        
        user = models.OnToOneField(settings.AUTH_USER_MODEL, primary_key=True)
        user_type = models.IntegerField(max_length=1, null=True,    choices=USER_TYPES)
        bio = models.CharField(max_length=200, blank=True, null=True)

        def __str__(self):
            return "{}:{:.20}".format(self.user, self.bio or "")

        class Meta:
            abstract = True


    class SuperHeroProfile(models.Model):
        origin = models.CharField(max_length=100, blank=True, null=True)

        class Meta:
            abstract = True


    class OrdinaryProfile(models.Model):
        address = models.CharField(max_length=200, blank=True, null=True)

        class Meta:
            abstract = True


    class Profile(SuperHeroProfile, OrdinaryProfile, BaseProfile):
        pass
```  

我们组织账户细节到多个抽象基类再到独立的关系中。`BaseProfile`类包含所有的不关心用户类型的公共账户细节。它也有一个`user_type`字段，它持续追踪用户的激活账户。  

`SuperHeroProfile`类和`OrdinaryProfile`类分别包含所有到超级英雄和非超级英雄特定账户细节。最后，所有这些基类的`profile`类创建了一个账户细节的超集。  

使用该方法时要主要的一些细节：  

    所有属于类的字段或者它抽象基类都必须是非空的，或者有一个默认值。  

    这个方法或许会为了每个用户而消耗更多的数据库，但是却带来极大的灵活性。  

    账户类型的激活和非激活字段都必须在模型外部是可管理的。  

    说到，编辑账户的表必须显示合乎目前激活用户类型的字段。  

## 模式-服务模式  
**问题**：模型会变得庞大而且不可管控。当一个模型不止实现一个功能时，测试和维护也会变得困难。  

**解决方法**：重构出一组相关方法到一个专用的`Service`对象中。  

### 问题细节
富模型，瘦视图是一个通常要告诉Django新手的格言。理论上，你的视图不应该包含任何其他表现逻辑。  

可是，随着时间的推移，代码段不能够放在任意地点，除非你打算将它们放进模型中。很快，模型会变成一个代码的垃圾场。  

下面是模型可以`Service`对象的征兆：  

    1. 与扩展能服务交互，例如web服务中，检查一个用户具有资格。  
    2. 帮助任务不会处理数据库，例如，生成一个短链接，或者针对用户的验证码。  
    3. 牵涉到一个短命的对象时不会存在数据库状态记录，烈日，创建一个AJAX调用的JSON响应。  
    4. 对于长时间运行的任务设计到了多实例，比如Celery任务。  

Django中的模型遵循着激活记录模式。理论上，它们同时封装应用罗即数据库访问。不过，要记得保持应用逻辑最小化。  

在测试时，如果我们发现没有对数据库建模的必要，甚至不会用到数据库，那么我们需要考虑把分解模型类。建议这种场合下使用`Service`对象。  


### 详解  
服务对象是封装一个`服务`或者和系统狡猾的普通而老旧的Python对象（POPOs）。它们通常保存在一个独立的称为`service.py`或者`utils.py`的文件。  

例如，像下面这样，检查一个web服务是否作为一个模型方法：  

```python
    class Profile(models.Model):
        ...

        def is_superhero(self):
            url = "http://api.herocheck.com/?q={0}".format(
                self.user.username
                )
            return webclient.get(url)
```  

该方法可以使用一个服务对象来重构：  

```python
    from .services import SuperHeroWebAPI

    def is_superhero(self):
        return SuperHeroWebAPI.is_superhero(self.user.username)
```  

现在服务对象可以定义在`services.py`中了：  

```python
    API_URL = "http://api.herocheck.com/?q={0}"

    class SuperHeroWebAPI:
        ...

        @staticmehtod
        def is_hero(username):
            url = API_URL.format(username)
            return webclient.get(url)
```  

多数情况下，`Service`对象的方法是无状态的，即，它们基于函数参数不使用任何的类属性来独自执行动作。因此，最好明确地把它们标记为静态方法（就像我们对`is_hero`所做的那样）。  

可以考虑把业务逻辑和域名逻辑从模型迁移到服务对象中去。这样，你可以在Django应用的外部很好使用它们。  
想象一下，由于业务的原因要依据某些用户的名字把这些要成为超级英雄的用户加进黑名单。我们的服务对象稍微改动以下就可以支持这个功能：  
```python
    class SuperHeroWebAPI:
        ...

    @staticmethod
    def is_hero(username):
        blacklist = set(["syndrome", "kcka$$", "superfake"])
        ulr = API_URL.format(username)
        return username not in blacklist and webclient.get(url)
```  

理论上，服务对象自包含的。这使它们不用建模——即数据库，也可以易于测试。它们也易于复用。  

Django中，耗时服务以Celery这样的异步任务队列方式执行。通常，`Service`对象以Celery任务的方式执行操作。这样的任务可以周期性地运行或者延迟运行。  

## 检索模式  
本节包含处理模型属性的访问，或者对模型执行查询。  


### 模式-属性字段  
**问题**：模型有以方法实现的属性。可是，这些属性不应该保存到数据库。  

***解决方案*：对这样的方法使用属性装饰器。

### 问题详情  
模型字段存储每个实例的属性，比如名，和姓，生日，等等。它们存储于数据库之中。可是，我们也需要访问某些派生的属性，比如一个完整的名字和年龄。  

它们可以轻易地计算数据库字段，因此它们不需要单独地存储。在某些情况下，它们可以成为一个检查所提供的年龄，会员积分，和激活状态是否合格的条件语句。  

简洁明了的实现这个方法是定义比如类似`get_age`这样函数：  

```python
    class BaseProfile(models.Model):
        birthdate = models.DateField()
        #...

        def get_age(self):
            today = datetime.date.today()
            return (today.year - self.birthdate.year) - int(
                (today.month, today.day) < (self.birthdate.month, self.birthdate.day)
                )
```  

调用`profile.get_age()`会通过计算调整过的月和日期所在的那个年份的不同来返回用户的年龄。  

不过，这样调用`profile.age`变得更加可读（和Python范）。  

### 详解  
Python类可以使用`property`装饰器把函数当作一个属性来使用。这样，Django模型也可以较好地利用它。替换前面那个例子中的函数：  
    
    @property
    def age(self):

现在我们可以用`profile.age`来访问用户的年龄。注意，函数的名称要尽可能的短。  

属性的一个重大缺陷是它对于ORM来说是不可访问的，就像模型的方法那样。你不能够在一个`Queryset`对象中使用它。例如，这么做是无效的，`Profile.objects.exlude(age__lt=18)。  

它也是一个定义一个属性来隐藏类内部细节的好主意。这也正式地称做*得墨忒耳定律*。简单地说，定律声明你应该只访问自己的直属成员或者“仅使用一个点号”。  

例如，最好是定义一个`profile.birthyear`属性，而不是访问`profile.birthdate.year`。这样，它有助于你隐藏`birthdate`字段的内在结构。  

>#### 提示
**最佳实践**  
*遵循得墨忒耳定律，并且访问属性时只使用点号*  

>该定律的一个不良反应是它导致在模型中有多个包装器属性被创建。这使模型膨胀并让它们变得难以维护。利用定律来改进你的模型API，减少模型间的耦合，在任何地方都是可行的。  

### 缓存特性  
每次我们调用一个属性时，就要重新计算函数。如果计算的代价很大，我们就想到了缓存结果。因此，下次访问属性，我们就拿到了缓存的结果。  
```python
    from django.utils.function import cached_property
    #...
    @cached_property
    def full_name(self):
        # 代价高昂的操作，比如，外部服务调用
        return "{0} {1}".format(self.firstname, self.lastname)
```  

缓存的值会作为Python实例的一部分而保存。只要实例一直存在，就会得到同样的返回值。  

对于一个保护性机制，你或许想要强制执行高昂代价操作以确保过期的值不会返回。如此情境之下，设置一个`cached=False`这样的关键字参数能够阻止返回缓存值。  

## 模式-定制模型管理器  
**问题**：某些模型的定义的查询被重复地访问，这彻彻底底的违反了DRY原则。  

**解决方案**：定义自定义的管理器以给常见的查询一个更有意义的名字。  

### 问题细节  
每一个Django的模型都有一个默认称做`objects`的管理器。调用`objects.all()`会返回数据库中的这个模型的所有条目。通常，我们只对所有条目的子集感兴趣。  

我们应用多种过滤器以找出所需的条目组。挑选它们的原则常常是我们的核心业务逻辑。例如，我们发现使用下面的代码可以通过public访问文章：  
```python

    public = Posts.objects.filter(privacy="public")
```

这个标准在未来或许会改变。我们或许也想要检查文章是否标记为编辑。这个改变或许如此：  
```python

    public = Posts.objects.filter(privacy=POST_PRIVACY.Public, draft=Flase)
```  

可是，这个改变需要使在任何地方都要用到公共文章。这让人非常沮丧。这仅需要一个定义这样常见地查询而无需“自我重复”。  

### 详解  
`Querysets`是一个极其有用的抽象概念。它们仅在需要时进行惰性查询。因此，通过链式方法（一个流畅的界面）构建更长的`Querysets`并不影响性能。  

事实上，应该更多的过滤会使结果数据集缩减。这样做通常可以减少结果的内存消耗。  

模型管理器是一个模型获取自身`Queryset`对象的便利接口。换句话来讲，它们有助于你使用Django的ORM访问潜在的数据库。事实上，`QuerySet`对象上管理器以一个非常简单的包装器实现。  

```python
    >>> Post.objects.filter(posted_by__username="a")
    [<Post:a: Hello World>， <Post:a: This is Private!>]

    >>> Post.objects.get_queryset().filter(posted_yb__username="a")
    [<Post:a: Hello World>, <Post:a: This is Private!>]
```  

默认的管理器由Django创建，`objects`有多种方法返回`Queryset`，比如`all`，`filter`或者`exclude`。可是，它们生成一个到数据库的低级API。  

定制管理器用于创建域名指定，更高级的API。这样不仅更加具有可读性而且通过实现细节减轻影响。因此，你能够在更高级的抽象概念上工作，紧密地模型化到域名。  

前面的公共文章例子可以轻松地转换到一个定制的管理器：  

```python
    # managers.py
    from django.db.models.query import Queryset


    class PostQuerySet(QuerySet):
        def public_posts(self):
            return self.filter(privacy="public")

    PostManager = PostQuerySet.as_manager
```  

这是一个在Django 1.7中从`QuerySet`对象创建定制管理器的捷径。不像前面的其他方法，这个`PostManager`对象像默认的`objects`管理器一样是链式的。  

如下所示，使用我们的定制管理器替换默认的`objects`管理器也是可行的：  

```python
    from .managers import PostManager

    class Post(Postable):
        ...
        objects = PostManager()
```

这样做，访问`public_posts`相当简单：  

```python
    public = Post.objects.public_posts()
```  

因此返回值是一个`QuerySet`，它们可以更进一步过滤：  

```python
    public_apology = Post.objects.public_posts().filter(
        message_startwith = "Sorry"
        )
```  

`QuerySets`由多个有趣的属性。在下一章，我们可以看看某些带有合并的`QuerySet`的常见地模式。  

### Querysets的组合动作  
事实上，对于它们的名字，`QuerySets`支持多组操作。为了说明，考虑包含用户对象的两个`QuerySets`：  

```python
    >>> q1 = User.objects.filter(username__in["a", "b", "c"])
    [<User:a>, <User:b>, <User:c>]
    >>> q2 = User.objects.filter(username__in["c", "d"])
    [<User:c>, <User:d>]
```  

对于一些组合操作可以执行以下动作：  

    *Union*：合并，移除重复动作。使用`q1`|`q2`获得[`<User: a>, <User: b>, <User: c>, <User: d>`]  

    *Intersection*：找出公共项。使用`q1`|`q2`获得[`<User: c>]  

    *Difference*：从第一个组中移除第二个组。该操作并不按逻辑来。改用`q1.exlude(pk__in=q2)获得［<User: a>, <User: b>］

同样的操作我们也可以用`Q`对象来完成：  

```python
    from django.db.models import Q

    # Union
    >>> User.objects.filter(Q(username__in["a", "b", "c"]) | Q(username__in=["c", "d"]))
    [`<User: a>, <User: b>, <User: c>, <User: d>`]

    # Intersection
    >>> User.objects.filter(Q(username__in["a", "b", "c"]) & Q(username__in=["c", "d"]))
    [<User: c>]

    # Difference
    >>> User.objects.filter(Q(username__in=["a", "b", "c"]) & ~Q(username__in=["c", "d"]))
    [<User: a>, <User: b>]
```  

注意执行动作所使用`&`（和）以及`~`（否定）的不同。`Q`对象是非常强大的，它可以用来构建非常复杂的查询。  

可是，`Set`虽类似但却不完美。`QuerySets`不像数学上的集合，那样按照顺序来。因此，在这一方面它们更接近Python的列表数据结构。  

### 链接多个Querysets  
目前为止，我们已经合并了属于相同基类的同类型`QuerySets`。可是，我们或许需要合并来自不同模型的`QuestSets`，并对它们执行操作。  

例如，一个用户的活动时间表包含了它们自身所有的按照反向时间顺序所发布的文章和评论。之前合并的`QuerySets`方法是不会起作用的。一个较为天真的做法是把它们转换到列表，连接并排列这个列表：  

```python
    >>> recent = list(posts)+list(comments)
    >>> sorted(recent, key=lambda e: e.modified, reverse=True)[:3]
    [<Post: user: Post1>, <Comment: user: Comment1>, <Post: user: Post0>]
```  

不幸的是，这个操作已经对惰性的`QuerySets`对象求值了。两个列表的内存使用算在一起可能很大内存开销。另外，转换一个庞大的`QuerySets`到列表是很慢很慢的。  

一个更好的解决方案是使用迭代器减少内存消耗。如下，使用`itertools.chain`方法合并多个`QuerySets`：  

```python
    >>> from itertools import chain
    >>> recent = chain(posts, comments)
    >>> sorted(recent, key=lambda e: e.modified, reverse=True)[:3]
```  

只要计算`QuerySets`，连接数据的开销都会非常搞。因此，重要的是，尽可能长的仅有的不对`QuerySets`求值的操作时间。  

>## 提示  
尽量延长`QuerySets`不求值的时间。  

## 迁移  
迁移让你改变模型时更有信心。说的Django 1.7，迁移已经是开发流程中基本的易于使用的一部分了。  

新的基本流程如下：  

    1. 第一次定义模型类的话，你需要运行：  

        python manage.py makemigrations <app_label>  


    2. 这将在`app/migrations/`文件夹内创建迁移脚本。  
    在同样的（开发）环境中运行以下命令：  
       
        python manage.py migrate <app_label>  
          

    3. 这将对数据库应用模型变更。有时候，遇到的问题有，处理默认值，重命名，等等。  
    
    4. 普及迁移脚本到其他的环境。通常，你的版本控制工具，例如，Git，会小心处理这事。当最新的源释出时，新的迁移脚本也会随之出现。  

    5. 在这些环境中运行下面的命令以应用模型的改变：  

        python manage.py migarte <app_label>  


    不论何时要将变更应用到模型，请重复以上1-5步骤。

如果你在命令里忽略了app标签，Django会在每一个app中发现未应用的变更并迁移它们。  


## 总结
模型设计要正确地操作很困难。它依然是Django开发的基础。本章，我们学习了使用模型时多个常见模式。每个例子中，我们都见到了建议方案的作用，以及多种折衷方案。  

这下一章，我们会用视图和URL配置来验证所遇到的常见设计模式。

--------------------
© Creative Commons BY-NC-ND 3.0   |   [我要订阅](https://github.com/cundi/Django-Design-Patterns-and-Best-Practices/subscription)   |   [我要捐助](https://github.com/cundi/Web.Development.with.Django.Cookbook/issues/3)

