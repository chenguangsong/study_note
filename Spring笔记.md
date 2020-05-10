# 1.IOC / DI

IOC ( Inversion of Control ) / DI (Dependency Injection)：控制反转。将对象的创建及依赖关系交给Spring统一控制，并不需要在在程序中通过New关键字创建。

##1.	装配```Bean```

###自动装配```bean```

自动装配```bean```包括两块内容：组件扫描和自动装配

- 组件扫描，借助```@ComponentScan```和```@Component```注解来完成。

  - ```@ComponentScan```用在Java配置类上，其作用是开启自动扫描功能。```@ComponentScan```有一个```basepackages```属性，可以指定需要扫描的包路径，其接收一个字符串或者字符串数组，当扫描的包路径只有一个时可以直接传入一个字符串，例如：```@ComponentScan("com.study") ```或者 ```@ComponentScan(basepackages="com.study")```。如果需要扫描的包是多个，只需要将```basepackages```属性设置为一个数组即可：```@ComponentScan(basepackages={"com.study.dao,com.stud.service"})```。

    也可以在XML中开启扫描功能，使用Spring Context命名空间的```<Context:component-scan base-package=""/>```元素来实现。

  - 设置了好需要扫描的包之后，就需要标记出包下需要扫描的类。通过在类上加```@Component```注解来告知Spring要为这个类创建```bean```。

- 自动装配，Spring在实例化```bean```时会对其依赖关系进行注入，只需要在属性或者方法上加```@AutoWired```注解，```@AutoWired```可以用在类的任何方法上，比如构造器、```Setter```方法等。Spring都会尝试去满足方法参数上所声明等依赖。假如有且只有一个```bean```满足依赖需求的话，那么这个```bean```就会被注入进来。但是如果没有匹配的```bean```或者有多个可以匹配的```bean```的话，Spring都会抛出异常。对于可能出现没有匹配```bean```的情况可以将```@AutoWired```的```required```属性设置为```false```，此时Spring将尝试自动装配，如果没有```bean```匹配，spring并不会抛出异常，而这个需要装配的属性处于未装配状态，其值为空，使用时要注意空判断，避免出现空指针异常。如果有个多个```bean```都满足条件，此时可以结合```@Qualifier```注解来解决。

```@AutoWired```是Spring特有的注解，还可以使用```@Inject```注解代替，```@Inject```注解源于Java依赖注入规范，Spring同时支持这两种方式，虽然它们之间也有些细微的差别，但在大多数情况下都可以相互替换。

###Java代码装配```bean```

自动装配是最简单且最方便使用，也是最推荐的一种方式。但是要将第三方库中的组件装配到自己的应用中，这种情况是没办法在它的类上添加```@Component```和```@AutoWired```注解的，这种情况就需要用到显示装配。

- 创建配置类，```@Configuration```表明这个类是一个配置类，其应包含Spring应用上下文中如何创建```bean```的细节。

  ```java
  package com.study.config;
  
  import org.springframework.context.annotation.Configuration;
  @Configuration
  public class JavaConfig {
  	
  }
  ```

- 声明简单的```bean```，```@Bean```注解会告诉Spring这个方法返回的对象需要注册到Spring应用上下文中，方法体包含了创建```bean```实例的逻辑。

  ```java
  package com.study.config;
  
  import com.study.pojo.User;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  @Configuration
  public class JavaConfig {
  	
  	@Bean
  	public User user(){
  		return new User();
  	}
  	
  }
  ```

  默认情况下，```bean```的的ID与带有```@Bean```注解的方法名是一样的，也可以通过```name```属性来指定一个名子:```@Bean(name = "")```。

- 实现注入

  ```java
  @Configuration
  public class JavaConfig {
  	@Bean
  	public Car car(){
  		return new Car();
  	}
  
  	@Bean()
  	public User user(){
  		return new User(car());
  	}
  }
  ```

  Spring中的```bean```都是单例的，Spring会拦截对```car()```的调用并返回的是Spring所创建的单例```bean```，这里并非每次调用```car()```都会创建一个新的```Car```对象。如果同时有两个对象依赖了```Car```实例，两个地方同时调用了```car()```方法，但实际上返回的都是同一个```Car```实例。

  还有另一种方式

  ```java
  @Bean
  public User user(Car car){
    return new User(car);
  }
  ```

  当Spring创建```User```对象时，它会自动装配一个```Car```到配置方法之中，并不需要明确引用```Car```的```@Bean```方法。这种方式并不要求将```Car```声明到同一个配置类当中，甚至不要求```Car```必须要在```JavaConfig```中声明，它可以通过组件扫描或者XML来配置。

###通过XML装配```Bean```

在XML配置中，需要创建一个XML文件，并且要以```<beans>```元素为根节点：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
                           http://www.springframework.org/schema/beans
                           https://www.springframework.org/schema/beans/spring-beans.xsd
                           ">
  <bean id="user" class="com.study.pojo.User"></bean>  
</beans>
```

要声明一个简单的```bean```只需要在XML文件中增加一个```<bean>```元素，这里的```class```属性要求是全限定类名。当Spring发现```<bean>```元素时，它会调用其默认构造器来创建```bean```。

构造器注入，利用```<constructor-arg>```元素，可以通过```id```将指定的```bean```实例利用构造器进行注入。

```xml
<bean id="car" class="com.study.pojo.Car"></bean>
<bean id="user" class="com.study.pojo.User">
  <constructor-arg ref="car"/>
</bean>
```

将字面量注入到构造器中，```name```属性指定了构造器中的属性名，```value```就是注入的值。

```xml
<bean id="user" class="com.study.pojo.User">
  <constructor-arg name="userName" value="张三"/>
  <constructor-arg name="age" value="21"/>
</bean>
```

设置属性，有时候并不需要将所有依赖都通过构造器来注入，如果依赖好几个都话，必须有一个全量依赖的构造器。这时可以使用设置属性的方式，也就是通过```Setter```方法注入，注意属性注入时，必须为该属性添加```Setter```方法

```java
<bean id="car" class="com.study.pojo.Car"></bean>
<bean id="user" class="com.study.pojo.User">
  <property name="car" ref="car"/>
</bean>
```

将字面量注入到属性中，也是利用```<property>```元素，只是值不在使用```ref```属性引用，而是```value```。

```java
<bean id="user" class="com.study.pojo.User">
  <property name="userName" value="张三"/>
  <property name="age" value="23"/>
</bean>
```

通过内嵌```<list>```元素将List集合注入，同样适合于通过构造器注入，也同样可以注入字面量集合。

```xml
<bean id="car1" class="com.study.pojo.Car"></bean>
<bean id="car2" class="com.study.pojo.Car"></bean>
<bean id="user" class="com.study.pojo.User">
  <property name="userName" value="张三"/>
  <property name="cars">
    <list>
      <ref bean="car1"/>
      <ref bean="car2"/>
    </list>
  </property>
</bean>
```

###混合装配

有时候我们会同时使用自动化(通过```@Component``` 和 ```@AutoWired```注解隐式完成自动扫描和装配)和显示配置(Java代码装配和XML文件装配)。其实，需要明确的是在自动装配时，它并不在意要装配的```bean```来自哪里，它会考虑Spring容器中的所有```bean```。

- 通过```@Import```注解将两外一个配置类导入到当前配置类

  ```java
  @Configuration
  @Import(CarConfig.class)
  public class UserConfig {
  	@Bean
  	public User user(){
  		return new User();
  	}
  }
  
  @Configuration
  public class CarConfig {
  	@Bean
  	public Car car(){
  		return new Car();
  	}
  }
  ```

  也可以通过一个更高层的配置类，将两个配置文件结合起来，而在这个更高层的配置类中不做任何初始化的逻辑，只是结合多个配置文件，其实就是利用了```@Import```可以传入一个数组的特性。

  ```java
  @Configuration
  @Import({UserConfig.class,CarConfig.class})
  public class JavaConfig {
  	
  }
  ```

- 通过```@ImportResource```注解可以在```JavaConfig```中引用XML配置，假设现在```Car```的配置放到了```applicationContext.xml```中进行配置，且该XML配置文件位于根类路径下。

  ```java
  @Configuration
  @Import({UserConfig.class,CarConfig.class})
  @ImportResource("classpath:applicationContext.xml")
  public class JavaConfig {
  
  }
  ```

- 同样，在XML配置中也可以将两个配置文件结合起来，它是通过```<import>```元素来实现的，例如在```user-config.xml```将```car-config.xml```结合

  ```xml
  <import resource="car-config.xml"/>
  ```

- 如果你现在维护的老的系统，之前所有的```bean```都配置在XML文件中，太多的配置已经接近无法控制，那么可以将新的配置放到```JavaConfig```中，在XML中将也可以```JavaConfig```引入

  ```xml
  <bean class="com.study.config.UserConfig"/>
  ```

以上，Spring中装配```bean```的三种方式，自动装配属于隐式的自动装配，而其他两个Java装配和XML装配属于显示装配。两种显示装配又都有两种注入方式，一种是构造器注入，一种是```Setter```方法注入，而隐式装配就是自动扫描自动装配，所以，有时候也说Spring装配```bean```的三种方式为自动扫描、构造器注入和```Setter```方法注入。

##2.	高级装配

###profile bean

####Java配置 profile bean

profile bean可以让Spring在实例化bean时作出选择，根据profile激活条件决定该创建哪个bean和不创建哪个bean。Spring并不是在构建的时候作出决策的，而是等到运行时才确定。profile bean 非常适合在实际项目中根据不同的环境配置不同的数据源的场景。

要使用profile，首先需要将所有不同的bean定义整理到一个或多个profile中，部署到不同的环境时确保对应的profile处于激活(active)状态。

在Java配置中可以使用@Profile指定某个bean属于哪个profile。@Profile注解可以使用在类级别上，表示该配置类创建的bean都属于同一个profile，不同的profile需要创建不同的JavaConfig。@Profile也可以与@Bean注解一同使用在方法级别上，这样就可以将两个不同profile bean放到同一个配置类中：

```java
@Configuration
public class JavaConfig {
	
	@Profile("dev")
	@Bean
	public User user(){
		return new User();
	}
	
	@Profile("prd")
	@Bean
	public Car car(){
		return new Car();
	}
	
}
```

####XML配置 profile bean

可以在XML配置文件中通过<beans>元素的profiles属性配置 profile bean，同一个XML配置文件所创建的bean都属于同一个profile。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
                           http://www.springframework.org/schema/beans
                           https://www.springframework.org/schema/beans/spring-beans.xsd"
       profile="dev">

  <bean id="car1" class="com.study.pojo.Car"></bean>
  <bean id="car2" class="com.study.pojo.Car"></bean>

  <bean id="user" class="com.study.pojo.User">
    <property name="userName" value="张三"/>
    <property name="cars">
      <list>
        <ref bean="car1"/>
        <ref bean="car2"/>
      </list>
    </property>
  </bean>
</beans>
```

还可以通过在<beans>元素中嵌套<beans>而不是为每个profile都创建一个XML，这样可以将不同的profile定义在同一个XML文件中。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
                           http://www.springframework.org/schema/beans
                           https://www.springframework.org/schema/beans/spring-beans.xsd">

  <beans profile="dev">
    <bean id="car1" class="com.study.pojo.Car" ></bean>
  </beans>
  <beans>
    <bean id="user" class="com.study.pojo.User"></bean>
  </beans>
</beans>
```

####激活 profile

Spring在确定哪个profile处于激活状态时，需要依赖两个独立的属性：spring.profiles.active 和 spring.profiles.default。Spring会优先选择active指定的profile，如果两个均没有设置的话，那就是没有激活状态的profile，只会创建那些没有定义在profile中的bean。可以同时激活多个profile，只需要列出多个profile名称并以逗号隔开即可。

其实有多种方式来设置两个属性：

- 作为DispatcherServlet的初始化参数；
- 作为Web应用的上下文参数；
- 作为JNDI条目；
- 作为环境变量；
- 作为JVM系统参数；
- 在集成测试类上使用@ActiveProfiles注解设置；

###条件化的 bean

@Conditional 注解与@Bean一起使用，@Conditonal注解接受一个Class参数，该参数类必须实现Condition接口，如果给定的条件返回true的话，就会创建这个bean，否则这个bean会被忽略。

```java
@Conditional(CarConditional.class)
@Bean
public Car car(){
  return new Car();
}

public class CarConditional implements Condition {
  @Override
  public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    return false;
  }
}
```

###自动装配歧义性问题

####设置首选的 bean

@Primary设置首选 bean，通过将@Primary注解与@Component或者@Bean注解组合使用，可以让Spring在自动注入遇到歧义时选择首选的 bean进行注入。该注解并不接受任何参数，需要时直接添加即可。

如果是XML配置的话，可以设置<bean>元素的primary属性为true来实现相同的效果：

```xml
<bean id="user" class="com.study.pojo.User" primary="true"></bean>
```

####限定符

@Qualifier注解可以与@AutoWired和@Inject注解协同使用，@Qualifier注解所设置的参数就是想要注入的 bean 的ID。而在Spring容器中，bean的ID是唯一的。

```java
@Bean
public Car car(){
  return new Car();
}

@Bean
public Car car2(){
  return new Car();
}

@Bean
@Qualifier("car2")
public User user(Car car){
  return new User(car);
}
```

##3. bean 的作用域

Spring定义类多种作用域，可以基于这些作用域创建 bean：

- 单例（Singleton）：在整个应用中，只创建 bean 的一个实例。
- 原型（Prototype）：每次注入或者通过Spring应用上下文获取的时候，都会创建一个新的 bean 实例。
- 会话（Session）：在Web应用中，为每个会话创建一个 bean 实例。
- 请求（Request）：在Web应用中，为每个请求创建一个 bean 实例。

单例是默认的作用域，可使用@Scope注解与@Component或@Bean注解一起使用来设置某个 bean 的作用域

```java
@Bean
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public Car car(){
  return new Car();
}

@Bean
@Scope("prototype")
public User user(){
  return new User();
}
```

如果是XML配置 bean 的话，可使用<bean>元素的scope属性来设置作用域

```xml
<bean id="user" class="com.study.pojo.User" primary="true" scope="prototype"></bean>
```

#2.AOP

Spring通过在目标对象的代理类中包裹切面，在运行期将切面织入到Spring管理的bean中，代理类封装了目标类，并拦截通知方法的调用，再把调用转发给真正的目标```bean```。当拦截到方法调用时，在调用目标```bean```方法之前，会执行切面逻辑。

###术语

#### 连接点（Join point）

连接点是指在应用执行过程中能够插入切面的一个点，可以是调用方法时、抛出异常时等。一般而言，连接点指的就是应用程序中的每个方法，每个方法就是一个连接点。Spring是在初始化时创建代理对象的。

####切点（Pointcut）

也称作切入点，顾名思义，就是在哪里进行切入从而将增强代码加入到业务主线中。切点有助于缩小切面需要通知的连接点的范围，通过明确类和方法名称或者使用正则表达式定义所匹配的类和方法来指定这些连接点即为切点。一个切点是一系列连接点的集合。切点用于准确定位应该在什么地方应用切面的通知。

#### 通知（Advice）

有了切点即已确定了在哪里进行切入，通知的意义是何时进行切入，分为5种类型的通知：

- 前置通知（Before）：在目标方法被调用之前调用通知功能；
- 后置通知（After）：在目标方法完成之后调用通知，但并不关心方法的输出是什么或者是否异常；
- 返回通知（After-returnning）：在目标方法成功执行之后调用通知；
- 异常通知（After-throwing）：在目标方法抛出异常后调用通知；
- 环绕通知（Around）：在目标方法调用之前和调用之后都调用通知；

#### 切面（Aspect）

切面就是通知和切点的结合。通知和切点共同定义了切面的全部内容——何时何处完成什么样的功能。一个切面就是被分离出来的一个功能，比如事务控制、日志打印。

#### 织入（Weaving）

织入是把切面应用到目标对象并创建新的代理对象的过程。Spring采用动态代理织入，就是在创建bean的代理对象时就将切面应用其中，而AspectJ采用编译期织入和类装载期织入。

### Java中使用注解创建切面

```java
@Aspect
public class MyAspectJ {

	// 定义切点
	@Pointcut("execution(** com.study.*(..))")
	public void pointCut(){}

	// 前置通知，在目标方法调用之前执行
	@Before("pointCut()")
	public void executeBefor(){
		System.out.println("前置通知执行……");
	}

	//异常通知，在目标方法执行异常后执行
	@AfterThrowing("pointCut()")
	public void executeError(){
		System.out.println("异常通知执行……");
	}

	//后置通知，在目标方法执行完后执行，无论是执行成功还是异常
	@After("pointCut()")
	public void executeAfter(){
		System.out.println("后置通知执行……");
	}

	//返回通知，在目标方法成功执行后执行
	@AfterReturning("pointCut()")
	public void executeSuccess(){
		System.out.println("返回通知执行……");
	}

	//环绕通知，在目标方法执行前、后都会调用
	@Around("pointCut()")
	public void aroundExcute(){
		System.out.println("环绕通知执行……");
	}
}

@Configuration
@ComponentScan
@EnableAspectJAutoProxy
public class JavaConfig {
}
```

- 通过在类上添加```@Aspect```注解，定义该类为一个切面。
- 通过```@Pointcut```注解定义了切点，其方法本身就是一个空，只供```@Pointcut```依附。
- 然后利用注解定义了五种通知类型。
- 最后需要在配置类上加```@EnableAspectJAutoProxy```注解，启动自动代理功能，否则当化```MyAspectJ.java```只是一个普通的POJO。并起不到切面的作用。

### XML中使用切面

```java
public class MyAspectJ {

  public void executeBefore(){
    System.out.println("前置通知执行……");
  }

  public void executeError(){
    System.out.println("异常通知执行……");
  }

  public void executeAfter(){
    System.out.println("后置通知执行……");
  }

  public void executeSuccess(){
    System.out.println("返回通知执行……");
  }

  public void aroundExcute(){
    System.out.println("环绕通知执行……");
  }
}
```

```xml
<!--启动AspectJ自动代理-->
<aop:aspectj-autoproxy/>

<!--声明 MyAspectJ bean-->
<bean id="myAspectJ" class="com.study.aop.MyAspectJ"/>

<aop:config>
  <aop:aspect ref="myAspectJ">
    <!--定义切点-->
    <aop:pointcut id="pointCut" expression="execution(* com.study.*(..))"/>
    <!--前置通知，在目标方法调用之前执行-->
    <aop:before method="executeBefore" pointcut-ref="pointCut"/>
    <!--异常通知，在目标方法执行异常后执行-->
    <aop:after-throwing method="executeError" pointcut-ref="pointCut"/>
    <!--后置通知，在目标方法执行完后执行，无论是执行成功还是异常-->
    <aop:after method="executeAfter" pointcut-ref="pointCut"/>
    <!--返回通知，在目标方法成功执行后执行-->
    <aop:after-returning method="executeSuccess" pointcut-ref="pointCut"/>
    <!--环绕通知，在目标方法执行前、后都会调用-->
    <aop:around method="aroundExcute" pointcut-ref="pointCut"/>
  </aop:aspect>
</aop:config>
```

XML配置切面的一个好处就是不需要对代码做任何修改，比如MyAspectJ是个原有的业务功能类，现在想把它作为一个切面，只需要在XML文件中对其进行声明和配置即可。而MyAspectJ现在除了是一个切面，可以完成切面工作之外还是一个普通的```bean```对象，还可以像使用普通对象一样使用其中的方法。

#3.Spring Bean的生命周期

1. 根据配置情况调用```bean```的构造方法或者工厂方法实例化``` bean```。
2. 完成```bean```中所有属性值的配置注入。
3. 如果Bean实现了```BeanNameAware```接口，则Spring调用```Bean```的```setBeanName()```方法传入当前```Bean```的```id```值。
4. 如果Bean实现了```BeanFactoryAware```接口，则Spring调用```Bean```的```setBeanFactory()```方法传入当前工厂实例的引用。
5. 如果Bean实现了```ApplicationContextAwrare```接口，则Spring调用```setApplicationContext()```方法传入当前```ApplicationContext```实例的引用。
6. 如果```BeanPostProcessor```和```Bean```关联，则Spring调用```BeanPostProcessor```接口的```Before```方法，在对象初始化前做预处理操作。
7. 如果Bean实现了```InitializingBean```接口，Spring将调用接口的```afterPropertiesSet()```方法。
8. 如果在配置文件通过```init-method```属性指定了初始化方法，则调用该初始化方法。
9. 如果```BeanPostProcessor```和```Bean```关联，则Spring将调用该接口的```After```方法，Spring AOP也是在此处生成的代理对象。此时，```Bean```已经完全创建好。
10. 如果```Bean```是单例的，则放入单例池中交给Spring进行管理，如果是原型类型的，则返回给调用者。
11. 如果```Bean```实现了```DisposableBean```，Spring会调用```destory()```方法将Spring ```Bean```销毁，如果在配置文件中通过属性```destroy-method```指定了销毁方法，则Spring将调用指定方法进行销毁。  
12. 如果

#4.事务

事务是指包含一个或多个对数据库的操作，这些操作要么全部成功，要么全部失败。常被用来保证数据的一致性。

事务为数据库提供了从失败中恢复数据，即使执行异常仍能保持一致的方法，并发访问数据库时，通过设置隔离级别防止彼此的操作互相干扰。事务是一次恢复和并发控制的基本单元。

### 事务隔离级别

#### 事务的四大特性

- 原子性

  事务作为一个整体被执行，事务中包含的对数据库的一系列操作，要么都成功，要么都失败。

- 一致性

  事务应保证数据库的状态从一个一致状态变为另一个一致状态。一致性状态的含义是数据库中的数据满足约束。

- 隔离性

  多个事务并发执行时，一个事务的执行不影响其他的事务。

- 持久性

  已提交的事务对数据的修改应该被永久保存到数据库中。

#### 事务的隔离级别

事务的隔离级别越高，越能保证数据的完整性和一致性。但对并发性能的影响越大。

标准的事务隔离级别包括：

- 读未提交(Read unCommited)：当前事务可以看到其他事务更改了但还没有提交的数据，存在脏读。
- 读提交(Read Commited)：当前事务在提交前可以看到其他事务已提交的数据。不可重复读。
- 可重复读(Repeatable Read)：一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。幻读，有可能在当前事务提交之前，别的事务对数据做了修改或者删除，但是当前事务还是能读取到且是旧的记录。
- 串行化(Serializable)：对于同一行记录，“写”会加“写锁”，“读”会加“读锁”。当出现读写冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行。

Spring中的事务是在容器将```Bean``` 实例化完成后，在```BenPostProcessor```后置处理器执行时通过动态代理生成代理对象来完成事务控制的。