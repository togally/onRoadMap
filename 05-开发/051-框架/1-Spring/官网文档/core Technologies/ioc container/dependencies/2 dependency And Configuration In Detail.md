## 直接指定值(straight value)
property标签或者construct-arg标签的value属性以字符串形式存储了其值，spring会利用conversion service相关服务去做类型转换，

将其转换为对应的值,同时在xml中为bean配置属性也是有多种方式的

· 普通方式
```xml
<beans>
    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <!-- results in a setDriverClassName(String) call -->
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
        <property name="username" value="root"/>
        <property name="password" value="misterkaoli"/>
    </bean>
</beans>
```

· p:XXX 简洁方式
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
	https://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
		destroy-method="close"
		p:driverClassName="com.mysql.jdbc.Driver"
		p:url="jdbc:mysql://localhost:3306/mydb"
		p:username="root"
		p:password="misterkaoli"/>

</beans>
```

· 借用value标签
```xml
<bean id="mappings"
	class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">

	<!-- typed as a java.util.Properties -->
	<property name="properties">
		<value>
			jdbc.driver.className=com.mysql.jdbc.Driver
			jdbc.url=jdbc:mysql://localhost:3306/mydb
		</value>
	</property>
</bean>
```

spring团队本身推荐使用<value></value>格式，因为该种方式是使用PropertyEditor机制将value标签内容转化为javaBean的属性值，该方法更高效

## idref标签的使用（The idRef element）
idref作用是将另一个bean的id注入
```xml
<beans>
    <bean id="theTargetBean" class="..."/>

    <bean id="theClientBean" class="...">
        <property name="targetName">
            <idref bean="theTargetBean"/>
        </property>
    </bean>
</beans>
```
上述xml文件完全等价于
```xml
<beans>
    <bean id="theTargetBean" class="..." />

    <bean id="client" class="...">
        <property name="targetName" value="theTargetBean"/>
    </bean>
</beans>
```
spring是推荐第一种的使用方式，因为第二种方式可能有认为犯错的风险，且不容易发现，而第一种方式在编译期，就会给你完成校验，确保theTargetBean一定存在。

## 依赖其他bean(references to others beans)
ref 标签是 <constructor-arg/> 或者 <property/> 标签的最后一个元素。该元素的作用是描述bean与bean之间的依赖关系，被依赖的bean会先被初始化。

所有的引用最终都是对另一个对象的引用，范围和验证取决于你是否通过bean和parent属性去指定id和name。
（ All references are ultimately a reference to another object. Scoping and validation depend on whether you specify the ID or name of the other object through the bean or parent attribute.）

下面给一个demo
父级容器
```xml
<!-- in the parent context -->
<bean id="accountService" class="com.something.SimpleAccountService">
	<!-- insert dependencies as required here -->
</bean>
```
子容器
```xml
<beans>
    <!-- in the child (descendant) context -->
    <bean id="accountService" class="org.springframework.aop.framework.ProxyFactoryBean"><!-- bean name is the same as the parent bean -->
    <property name="target">
        <ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
    </property>
    <!-- insert other configuration and dependencies as required here -->
</bean>
</beans>
```