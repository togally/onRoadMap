## 基础流程
![基础流程](../img/mybatis基础流程.png)

首先我们需要从给定的配置文件路径中去解析出mapper与sql和对象的映射规则，然后需要创建好代理类。

然后我们需要将创建好的代理类注册到配置类中方便读取

使用的时候会根据调用的mapper获取对应的代理类去获得映射的sql

## 解析注册流程
![解析以及注册流程](../img/解析以及注册流程.png)
首先当我们去构建sqlSessionFactory的时候，也就是调用SqlSessionFactory.build(Reader)方法时

系统会调用Resources类下的方法getResourceAsReader获取mapper.xml的读取类，并生成reader

然后build方法内部会去实例化XmlConfigBuilder类，并调用parse()方法执行解析

解析时会去读取在xml中所定义的包路径地址，并调用MapperRegistry也就是mapper注册器的addMappers

注册器会去扫描包路径，将路径下的mapper生成对应的MapperProxyFactory并注册到注册器中

当我们完成注册之后就会利用解析完成后的Configuration配置去构造一个SqlSessionFactory

## 使用流程
![mapper调用流程](../img/mapper调用流程.png)
整个过程实际上是基于代理的

使用的前提时需要构造好sqlSessionFactory对象

接着我们就可以利用sqlSessionFactory去开启一个session，并获取到对象mapper的代理类

每个mapper接口文件都有一个MapperProxyFactory,是在解析时候注册到MapperRegistry中的

Proxy中中包装了MapperMethod，执行invoke方法方法时会调用MapperMethod 的execute方法

每个MapperMethod记录了包装方法的类型，根据方法类型不同去调用不同的sqlSession方法（insert/update/delete/select等）

此时在sqlSession内部会根据configuration去获取MappedStatement(映射语句),并交由executor去执行一个不同的操作流程

基本上来说executor会先获取connect,并由connect和MappedStatement共同构建Statement,并执行Statement.execute();