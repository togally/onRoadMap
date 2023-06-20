## 依赖注入概述
依赖注入指的是一个过程，指的是对象被创建时，容器根据对象本身的依赖关系来进行属性注入的过程，这个过程也叫控制反转。

对象的依赖关系可以通过构造器的参数,工厂方法参数,或者（对象本身、工厂方法的返回对象）自己本身的属性。

使用依赖注入会是的代码变得很干净，解耦更彻底，对象本身无需知道自己的依赖关系，不用知道自己某个关系的位置，类型等信息。

依赖注入主要分两部分： 构造器注入或者是setter方法注入

## 基于构造器的注入
 基于构造器的注入是根据构造器的参数来完成的，每个参数都是一个依赖关系，容器会根据构造器提供的参数来进行依赖的注入，下面是一个例子

```java
public class SimpleMovieLister {

	// the SimpleMovieLister has a dependency on a MovieFinder
	private final MovieFinder movieFinder;

	// a constructor so that the Spring container can inject a MovieFinder
	public SimpleMovieLister(MovieFinder movieFinder) {
		this.movieFinder = movieFinder;
	}

	// business logic that actually uses the injected MovieFinder is omitted...
}
```
#### 构造参数解析
    构造函数注入是基于构造器的参数来进行匹配的，如果bean定义中的构造器参数不存在歧义那么，bean定义中的顺序就是配置中提供的构造器参数顺序
```xml
<beans>
	<bean id="beanOne" class="x.y.ThingOne">
		<constructor-arg ref="beanTwo"/>
		<constructor-arg ref="beanThree"/>
	</bean>

	<bean id="beanTwo" class="x.y.ThingTwo"/>

	<bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```

```java
package x.y;

public class ThingOne {

    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        // ...
    }
}
```
匹配简单类型的属性时，指定value后害需要显示的指定type，因为spring并不能知道值得类型
```xml
<bean id="exampleBean" class="examples.ExampleBean">
	<constructor-arg type="int" value="7500000"/>
	<constructor-arg type="java.lang.String" value="42"/>
</bean>
```

当构造器顺序由歧义的时候可以通过index属性来指定
```xml
<bean id="exampleBean" class="examples.ExampleBean">
	<constructor-arg index="0" value="7500000"/>
	<constructor-arg index="1" value="42"/>
</bean>
```
也可以通过参数的名称来消去歧义
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```
使用name来进行标识需要配合注解
```java
package examples;

public class ExampleBean {

	// Fields omitted

	@ConstructorProperties({"years", "ultimateAnswer"})
	public ExampleBean(int years, String ultimateAnswer) {
		this.years = years;
		this.ultimateAnswer = ultimateAnswer;
	}
}
```

## 基于setter方法注入
setter注入是再无参构造或者有参构造之后,基于set方法注入
```java
public class SimpleMovieLister {

	// the SimpleMovieLister has a dependency on the MovieFinder
	private MovieFinder movieFinder;

	// a setter method so that the Spring container can inject a MovieFinder
	public void setMovieFinder(MovieFinder movieFinder) {
		this.movieFinder = movieFinder;
	}

	// business logic that actually uses the injected MovieFinder is omitted...
}
```
spring application支持构造器注入和setter注入。我们通过构造器注入属性之后也可以通过beanDefinition结合PropertyEditor实体去将
一个属性转化为类一种类型

## 关于依赖的处理
对于依赖的处理由如下绩点
* 容器通过配置元数据来进行创建和初始化，配置元数据基于xml，java代码，注解来进行描述
* 每个bean的依赖关系可以通过属性、构造参数、静态工厂方法的参数（如果你用它去替代正常的构造方法）
* 构造方法中的每个入餐都是实际定义的值或者是另一个bean的引用
* 默认情况下，每个属性或者构造参数都由spring将字符串值转换为实际对应的参数值

spring在创建容器是就会校验每个bean的配置，但是并不会真正的给他们set值，当去获取bean的时候才会真正的去set值。作用域为默认（pre-instantiated）、
singleton（单例）会在容器创建后创建。除此之外，bean会在第一次需要获取他的时候来创建。随着bean的依赖被创建，以及bean的依赖的依赖被创建
和分配（等等），bean创建可能会导致bean图的创建，

### 循环依赖问题
```text
    当我们使用构造器注入的时候，可能会出现循环依赖的问题。
    所谓循环依赖就是A类与B类的构造器互为对方的参数，A构造器以B类作为入参，B构造器以A作为入参。当产生这种情况的时候会抛出BeanCurrentlyInCreationException异常
    // TODO 为什么构造器方法存在循环依赖问题,而set方法不存在
    避免该问题我们可以使用setter方法注入，而不是构造器注入。
    spring通过三级缓存机制来解决循环依赖问题。通过将半成品的bean注入到另一个bean中从而打破鸡生蛋，蛋生鸡的循环。
    // TODO 三级缓存代码层面描述
```

## spring注入的一些例子
### set方法注入
set方法注入就是在xml中声明属性，并且在java代码中提供set方法
```XML
<beans>
    <bean id="exampleBean" class="examples.ExampleBean">
        <!-- setter injection using the nested ref element -->
        <property name="beanOne">
            <ref bean="anotherExampleBean"/>
        </property>

        <!-- setter injection using the neater ref attribute -->
        <property name="beanTwo" ref="yetAnotherBean"/>
        <property name="integerProperty" value="1"/>
    </bean>

    <bean id="anotherExampleBean" class="examples.AnotherBean"/>
    <bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
</beans>
```
### 构造器注入
```XML
<beans>
    <bean id="exampleBean" class="examples.ExampleBean">
        <!-- constructor injection using the nested ref element -->
        <constructor-arg>
            <ref bean="anotherExampleBean"/>
        </constructor-arg>

        <!-- constructor injection using the neater ref attribute -->
        <constructor-arg ref="yetAnotherBean"/>

        <constructor-arg type="int" value="1"/>
    </bean>

    <bean id="anotherExampleBean" class="examples.AnotherBean"/>
    <bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
</beans>
```

### 工厂方法的入参
```xml
<beans>
    <bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
        <constructor-arg ref="anotherExampleBean"/>
        <constructor-arg ref="yetAnotherBean"/>
        <constructor-arg value="1"/>
    </bean>

    <bean id="anotherExampleBean" class="examples.AnotherBean"/>
    <bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
</beans>
```

```java
public class ExampleBean {

	// a private constructor
	private ExampleBean(...) {
		...
	}

	// a static factory method; the arguments to this method can be
	// considered the dependencies of the bean that is returned,
	// regardless of how those arguments are actually used.
	public static ExampleBean createInstance (
		AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

		ExampleBean eb = new ExampleBean (...);
		// some other operations...
		return eb;
	}
}
```
[原文](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html#beans-constructor-injection)
