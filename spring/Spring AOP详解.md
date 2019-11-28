# Spring AOP详解

参考博文地址：http://www.cnblogs.com/xrq730/p/4919025.html 

​						    https://blog.csdn.net/xiaoxian8023/article/details/17285809 

## AOP原理

AOP（Aspect Oriented Programming），即面向切面编程，可以说是OOP（Object Oriented Programming，面向对象编程）的补充和完善。OOP引入封装、继承、多态等概念来建立一种对象层次结构，用于模拟公共行为的一个集合。不过OOP允许开发者定义**纵向**的关系，但并不适合定义横向的关系，例如日志功能。日志代码往往横向地散布在所有对象层次中，而与它对应的对象的核心功能毫无关系对于其他类型的代码，如安全性、异常处理和透明的持续性也都是如此，这种散布在各处的无关的代码被称为横切（cross cutting），在OOP设计中，它导致了大量代码的重复，而不利于各个模块的重用。

AOP技术恰恰相反，它利用一种称为"横切"的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其命名为"Aspect"，即切面。所谓"切面"，简单说就是那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块之间的耦合度，并有利于未来的可操作性和可维护性。

使用"横切"技术，AOP把软件系统分为两个部分：**核心关注点**和**横切关注点**。业务处理的主要流程是核心关注点，与之关系不大的部分是横切关注点。横切关注点的一个特点是，他们经常发生在核心关注点的多处，而各处基本相似，比如权限认证、日志、事物。AOP的作用在于分离系统中的各种关注点，将核心关注点和横切关注点分离开来。



##  **AOP核心概念** 

1、横切关注点

对哪些方法进行拦截，拦截后怎么处理，这些关注点称之为横切关注点

2、切面（aspect）

类是对物体特征的抽象，切面就是对横切关注点的抽象

3、连接点（joinpoint）

被拦截到的点，因为Spring只支持方法类型的连接点，所以在Spring中连接点指的就是被拦截到的方法，实际上连接点还可以是字段或者构造器

4、切入点（pointcut）

对连接点进行拦截的定义

5、通知（advice）

所谓通知指的就是指拦截到连接点之后要执行的代码，通知分为前置、后置、异常、最终、环绕通知五类

6、目标对象

代理的目标对象

7、织入（weave）

将切面应用到目标对象并导致代理对象创建的过程

8、引入（introduction）

在不修改代码的前提下，引入可以在**运行期**为类动态地添加一些方法或字段



##  **Spring对AOP的支持** 

**Spring中AOP代理由Spring的IOC容器负责生成、管理，其依赖关系也由IOC容器负责管理**。因此，AOP代理可以直接使用容器中的其它bean实例作为目标，这种关系可由IOC容器的依赖注入提供。Spring创建代理的规则为：

1、**默认使用Java动态代理来创建AOP代理**，这样就可以为任何接口实例创建代理了

2、**当需要代理的类不是代理接口的时候，Spring会切换为使用CGLIB代理**，也可强制使用CGLIB

AOP编程其实是很简单的事情，纵观AOP编程，程序员只需要参与三个部分：

1、定义普通业务组件

2、定义切入点，一个切入点可能横切多个业务组件

3、定义增强处理，增强处理就是在AOP框架为普通业务组件织入的处理动作

所以进行AOP编程的关键就是定义切入点和定义增强处理，一旦定义了合适的切入点和增强处理，AOP框架将自动生成AOP代理，即：**代理对象的方法=增强处理+被代理对象**的方法。

下面给出一个Spring AOP的application-config.xml.xml文件模板

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-4.2.xsd">
            
</beans>
```



##  Spring AOP实例之xml配置 

注意一下，在讲解之前，说明一点：使用Spring AOP，要成功运行起代码，只用Spring提供给开发者的jar包是不够的，请额外上网下载两个jar包：

1、aopalliance.jar

2、aspectjweaver.jar

开始讲解用Spring AOP的XML实现方式，先定义一个接口：

```java
public interface HelloWorld
{
    void printHelloWorld();
    void doPrint();
}
```

 定义两个接口实现类： 

```java
public class HelloWorldImpl1 implements HelloWorld
{
    public void printHelloWorld()
    {
        System.out.println("Enter HelloWorldImpl1.printHelloWorld()");
    }
    
    public void doPrint()
    {
        System.out.println("Enter HelloWorldImpl1.doPrint()");
        return ;
    }
}
```

```java
public class HelloWorldImpl2 implements HelloWorld
{
    public void printHelloWorld()
    {
        System.out.println("Enter HelloWorldImpl2.printHelloWorld()");
    }
    
    public void doPrint()
    {
        System.out.println("Enter HelloWorldImpl2.doPrint()");
        return ;
    }
}
```

 横切关注点，这里是打印时间： 

```java
public class TimeHandler
{
    public void printTime()
    {
        System.out.println("CurrentTime = " + System.currentTimeMillis());
    }
}
```

 有这三个类就可以实现一个简单的Spring AOP了，看一下application-config.xml的配置： 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-4.2.xsd">
        
        <bean id="helloWorldImpl1" class="com.xrq.aop.HelloWorldImpl1" />
        <bean id="helloWorldImpl2" class="com.xrq.aop.HelloWorldImpl2" />
        <bean id="timeHandler" class="com.xrq.aop.TimeHandler" />
        
        <aop:config>
            <aop:aspect id="time" ref="timeHandler">
                <aop:pointcut id="addAllMethod" expression="execution(* com.xrq.aop.HelloWorld.*(..))" />
                <aop:before method="printTime" pointcut-ref="addAllMethod" />
                <aop:after method="printTime" pointcut-ref="addAllMethod" />
            </aop:aspect>
        </aop:config>
</beans>
```

 写一个main函数调用一下： 

```java
public static void main(String[] args)
{
    ApplicationContext ctx = 
            new ClassPathXmlApplicationContext("aop.xml");
        
    HelloWorld hw1 = (HelloWorld)ctx.getBean("helloWorldImpl1");
    HelloWorld hw2 = (HelloWorld)ctx.getBean("helloWorldImpl2");
    hw1.printHelloWorld();
    System.out.println();
    hw1.doPrint();
    
    System.out.println();
    hw2.printHelloWorld();
    System.out.println();
    hw2.doPrint();
}
```

 运行结果为： 

```
CurrentTime = 1446129611993
Enter HelloWorldImpl1.printHelloWorld()
CurrentTime = 1446129611993

CurrentTime = 1446129611994
Enter HelloWorldImpl1.doPrint()
CurrentTime = 1446129611994

CurrentTime = 1446129611994
Enter HelloWorldImpl2.printHelloWorld()
CurrentTime = 1446129611994

CurrentTime = 1446129611994
Enter HelloWorldImpl2.doPrint()
CurrentTime = 1446129611994
```



# Spring Aop实例之AspectJ注解配置

 接口和两个实现类代码参考XML配置

 Aspect类 分享：

```java
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.DeclareParents;
import org.aspectj.lang.annotation.Pointcut;

/**
    测试after,before,around,throwing,returning Advice.
    @author Admin
*/
@Aspect
public class AspceJAdvice {

    /**
        Pointcut
        定义切面 ，切面的名称为aspectjMethod()
        此方法没有返回值和参数该方法就是一个标识，不进行调用
        */
    @Pointcut("execution(* com.xrq.aop.HelloWorld.*(..))")
    private void aspectjMethod(){};
    
    /** 
        Before
        在核心业务执行前执行，不能阻止核心业务的调用。
        @param joinPoint 切点即某个业务方法  
        */  
    @Before("aspectjMethod()")  
    public void beforeAdvice(JoinPoint joinPoint) {  
        System.out.println("-----beforeAdvice().invoke-----");
        System.out.println(" 此处意在执行核心业务逻辑前，做一些安全性的判断等等");
        System.out.println("目标方法名为:" + joinPoint.getSignature().getName());
        System.out.println("目标方法声明类型:" + Modifier.toString(
            joinPoint.getSignature().getModifiers()));
        System.out.println("-----End of beforeAdvice()------");
    }
    
    /** 
        After 
        核心业务逻辑退出后（包括正常执行结束和异常退出），执行此Advice
        @param joinPoint
        */
	@After(value = "aspectjMethod()")  
	public void afterAdvice(JoinPoint joinPoint) {  
    	System.out.println("-----afterAdvice().invoke-----");
    	System.out.println(" 此处意在执行核心业务逻辑之后，做一些日志记录操作等等");
    	System.out.println(" 可通过joinPoint来获取所需要的内容");
    	System.out.println("-----End of afterAdvice()------");
    }  
    
	/** 
       Around环绕方法,可自定义目标方法执行的时机 
       手动控制调用核心业务逻辑，以及调用前和调用后的处理,
       注意：当核心业务抛异常后，立即退出，转向AfterAdvice
       执行完AfterAdvice，再转到ThrowingAdvice
       @param pjp
       @throws Throwable
       @return Object
       */ 
	@Around(value = "aspectjMethod()")  
	public Object aroundAdvice(ProceedingJoinPoint pjp) {  
        Object retVal = null;
		System.out.println("-----aroundAdvice().invoke-----");
		System.out.println(" 此处可以做类似于Before Advice的事情");
        //调用核心逻辑
        try {
        	retVal = pjp.proceed();
            } catch (Throwable e) {
            //异常通知
            System.out.println("执行目标方法异常后...");
            throw new RuntimeException(e);
        }
        //后置通知
        System.out.println(" 此处可以做类似于After Advice的事情");
        System.out.println("-----End of aroundAdvice()------");
        return retVal;
	}  

	/** 
        AfterReturning 
        核心业务逻辑调用正常退出后，不管是否有返回值，正常退出后，均执行此Advice
        @param joinPoint
        @param retVal
	*/ 
	@AfterReturning(value = "aspectjMethod()", returning = "retVal")  
	public void afterReturningAdvice(JoinPoint joinPoint, String retVal) {  
       System.out.println("-----afterReturningAdvice().invoke-----");
       System.out.println("Return Value: " + retVal); 
       System.out.println(" 此处可以对返回值做进一步处理");
       System.out.println(" 可通过joinPoint来获取所需要的内容");
       System.out.println("-----End of afterReturningAdvice()------");
	}
    
	/**
       核心业务逻辑调用异常退出后，执行此Advice，处理错误信息
       注意：执行顺序在Around Advice之后
       @param joinPoint
       @param ex
	*/
	@AfterThrowing(value = "aspectjMethod()", throwing = "ex")  
	public void afterThrowingAdvice(JoinPoint joinPoint, Exception ex) {  
        System.out.println("-----afterThrowingAdvice().invoke-----");
        System.out.println(" 错误信息："+ex.getMessage());
        System.out.println(" 此处意在执行核心业务逻辑出错时，捕获异常，并可做一些日志记录操作等等");
        System.out.println(" 可通过joinPoint来获取所需要的内容");
        System.out.println("-----End of afterThrowingAdvice()------");  
	}  
    
}
```


 application-config.xml , 只需要配置业务逻辑bean和Aspect bean，并启用Aspect注解即可 :

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
	     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	     xmlns:aop="http://www.springframework.org/schema/aop"
	     xmlns:tx="http://www.springframework.org/schema/tx"
	     xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.0.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.0.xsd
           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.0.xsd">
	<!-- 启用AspectJ对Annotation的支持 -->       
    <aop:aspectj-autoproxy/>           

    <bean id="helloWorldImpl1" class="com.xrq.aop.HelloWorldImpl1" />
    <bean id="helloWorldImpl2" class="com.xrq.aop.HelloWorldImpl2" />
    <bean id="aspcejHandler" class="com.xrq.aop.AspceJAdvice"/>

</beans>
```





