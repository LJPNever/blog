---
title: Springboot 启动相关
date: 2019-04-01 09:19:58
tags:
---

### Springboot 启动相关

<!--more-->

https://www.cnblogs.com/xinzhao/p/5551828.html

入口：

```java
@SpringBootApplication
public class ServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServerApplication.class,args);
    }
}

```

**进入run方法**，首先创建了SpringApplication对象，并调用该对象的run方法

```java
public static ConfigurableApplicationContext run(Object source, String... args) {
        return run(new Object[]{source}, args);
    }
public static ConfigurableApplicationContext run(Object[] sources, String[] args) {
        return (new SpringApplication(sources)).run(args);
    }
```

创建完**SpringApplication**后，构造函数中调用了**initialize**方法，初始化每一个变量

```java
  public SpringApplication(Object... sources) {
        this.bannerMode = Mode.CONSOLE;
        this.logStartupInfo = true;
        this.addCommandLineProperties = true;
        this.headless = true;
        this.registerShutdownHook = true;
        this.additionalProfiles = new HashSet();
      //我们传进来的参数
        this.initialize(sources);
    }
```

```java
   private void initialize(Object[] sources) {
       //为成员变量sources赋值 
        if (sources != null && sources.length > 0) {
            this.sources.addAll(Arrays.asList(sources));
        }
       //为接下来每一个变量赋值
        this.webEnvironment = this.deduceWebEnvironment();                              this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));
        this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
        this.mainApplicationClass = this.deduceMainApplicationClass();
    }
```

1. webEnvironment是一个boolean 类型的值，用来表示当前的应用程序是不是一个web应用程序

```java
//通过在classpath 中查找是否存在 WEB_ENVIRONMENT_CLASSES这个数组所包含的类，如果存在那么当前程序是一个web 程序
private boolean deduceWebEnvironment() {
        String[] var1 = WEB_ENVIRONMENT_CLASSES;
        int var2 = var1.length;

        for(int var3 = 0; var3 < var2; ++var3) {
            String className = var1[var3];
            if (!ClassUtils.isPresent(className, (ClassLoader)null)) {
                return false;
            }
        }

        return true;
    }
```

1. initializers成员变量，是一个ApplicationContextInitializer类型对象的集合。 顾名思义，ApplicationContextInitializer是一个可以用来初始化ApplicationContext的接口

```java
//调用getSpringFactoriesInstances(ApplicationContextInitializer.class)，来获取ApplicationContextInitializer类型对象的列表。
public void setInitializers(Collection<? extends ApplicationContextInitializer<?>> initializers) {
        this.initializers = new ArrayList();
        this.initializers.addAll(initializers);
    }



   private <T> Collection<? extends T> getSpringFactoriesInstances(Class<T> type) {
        return this.getSpringFactoriesInstances(type, new Class[0]);
    }

    private <T> Collection<? extends T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
        ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
        Set<String> names = new LinkedHashSet(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
        List<T> instances = this.createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
        AnnotationAwareOrderComparator.sort(instances);
        return instances;
    }
```

通过SpringFactoriesLoader.loadFactoryNames(type, classLoader)来获取所有Spring  Factories的名字，然后调用createSpringFactoriesInstances方法根据读取到的名字创建对象，最后将创建好的对象列表排序返回。在loadFactoryNames中主要为  classLoader.getResources("META-INF/spring.factories") ，则从中读取最近的ConfigurationWarningsApplicationContextInitializer，ContextIdApplicationContextInitializer，DelegatingApplicationContextInitializer，ServerPortInfoApplicationContextInitializer这四个类的名字

之后创建ApplicationContextInitializer实例，为4个组成的list

1. listeners成员变量，是一个ApplicationListener<?>类型对象的集合。可以看到获取该成员变量内容使用的是跟成员变量initializers一样的方法，只不过传入的类型从ApplicationContextInitializer.class变成了ApplicationListener.class。

```java
  public void setListeners(Collection<? extends ApplicationListener<?>> listeners) {
        this.listeners = new ArrayList();
        this.listeners.addAll(listeners);
    }
```

1. 在deduceMainApplicationClass方法中，通过获取当前调用栈，找到入口方法main所在的类，并将其复制给SpringApplication对象的成员变量mainApplicationClass

```java
  public void setListeners(Collection<? extends ApplicationListener<?>> listeners) {
        this.listeners = new ArrayList();
        this.listeners.addAll(listeners);
    }
```

初始化完毕后，将调用run 方法，

```java
 public ConfigurableApplicationContext run(String... args) 
```

1. 设置系统属性java.awt.headless

```java
this.configureHeadlessProperty();
```

1. 在创建和更新ApplicationContext前后分别调用了started方法和finished方法, 并在创建和刷新ApplicationContext时，将listeners作为参数传递到了createAndRefreshContext方法中，以便在创建和刷新ApplicationContext的不同阶段，调用listeners的相应方法以执行操作。所以，所谓的SpringApplicationRunListeners实际上就是在SpringApplication对象的run方法执行的不同阶段，去执行一些操作，加载SpringBoot配置环境(ConfigurableEnvironment)，如果是通过web容器发布，会加载StandardEnvironment，其最终也是继承了ConfigurableEnvironment

```java
SpringApplicationRunListeners listeners = this.getRunListeners(args);
```

另外EventPublishingRunListener在对象初始化时，将SpringApplication对象的成员变量listeners全都保存下来，然后在自己的public方法被调用时，发布相应的事件，或执行相应的操作。可以说这个RunListener是在SpringApplication对象的run方法执行到不同的阶段时，发布相应的event给SpringApplication对象的成员变量listeners中记录的事件监听器。

1. 接下来 创建并刷新ApplicationContext，并最后调用afterRefresh方法在刷新之后做一些操作

```java
context = this.createApplicationContext();
            new FailureAnalyzers(context);
            this.prepareContext(context, environment, listeners, applicationArguments, printedBanner);
            this.refreshContext(context);
            this.afterRefresh(context, applicationArguments);
            listeners.finished(context, (Throwable)null);

```

然后ApplicationListener进行响应事件，其中 LoggingApplicationListener响应该事件，并对在ApplicationStarted时加载的LoggingSystem做一些初始化工作，之后会打印banner

创建ApplicationContext，当检测到本次程序是一个web应用程序（成员变量webEnvironment为true）的时候，就加载类DEFAULT_WEB_CONTEXT_CLASS，否则的话加载DEFAULT_CONTEXT_CLASS。



##### Application run关键：

1. 创建了应用的监听器，并开始监听

   ```
   SpringApplicationRunListeners listeners = this.getRunListeners(args);
   listeners.starting();
   ```

2. 加载了SpringBoot配置环境（ConfigurableEnvironment），如果当前是通过web 容器发布（通过成员变量webEnvironment判断），会加载StandardEnvironment，最终也是继承了ConfigurableEnvironment。

```java
ConfigurableEnvironment environment = this.prepareEnvironment(listeners, applicationArguments);
Banner printedBanner = this.printBanner(environment);
```

1. 配置环境(Environment)加入到监听器对象中(SpringApplicationRunListeners)
2. 创建run方法的返回对象，ConfigurableApplicationContext(应用配置上下文)，该方法会先获取显式设置的应用上下文(applicationContextClass)，如果不存在，再加载默认的环境配置（通过是否是web environment判断），默认选择AnnotationConfigApplicationContext注解上下文（通过扫描所有注解类来加载bean），最后通过BeanUtils实例化上下文对象，并返回，ConfigurableApplicationContex

```java
protected ConfigurableApplicationContext createApplicationContext() {
        Class<?> contextClass = this.applicationContextClass;
        if (contextClass == null) {
            try {
                contextClass = Class.forName(this.webEnvironment ? "org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext" : "org.springframework.context.annotation.AnnotationConfigApplicationContext");
            } catch (ClassNotFoundException var3) {
                throw new IllegalStateException("Unable create a default ApplicationContext, please specify an ApplicationContextClass", var3);
            }
        }

        return (ConfigurableApplicationContext)BeanUtils.instantiate(contextClass);
    }
```

1. 回到run 中，prepareContext方法将listeners、environment、applicationArguments、banner等重要组件与上下文对象关联

```java
  this.prepareContext(context, environment, listeners, applicationArguments, printedBanner);
```

1. 接下来的refreshContext(context)方法(初始化方法如下)将是实现spring-boot-starter-*(mybatis、redis等)自动化配置的关键，包括spring.factories的加载，bean的实例化等核心工作

```java
  protected void refresh(ApplicationContext applicationContext) {
        Assert.isInstanceOf(AbstractApplicationContext.class, applicationContext);
        ((AbstractApplicationContext)applicationContext).refresh();
    }
```

配置结束后，Springboot做了一些基本的收尾工作，返回了应用环境上下文。回顾整体流程，Springboot的启动，主要创建了配置环境(environment)、事件监听(listeners)、应用上下文(applicationContext)，并基于以上条件，在容器中开始实例化我们需要的Bean，至此，通过SpringBoot启动的程序已经构造完成。









Springboot 在启动时，加载了@SpringBootApplication注解主配置类，在注解主配置类里面的主要功能是为speingboot 开启了一个 @EnableAutoConfiguration 注解的自动配置功能，EnableAutoConfiguration 主要利用了**AutoConfigurationImportSelector**来导入一些组件，主要是 扫描 META-INF/spring.factories文件，从中获取要交给spring的所有组件







#### Spring IOC；

ico的实现类，顶层为BeanFactory接口，有一个为ApplicationContext的子接口，其实现类都为IOC容器，因为 ApplicationContext功能强大，可以通过配置文件来实现功能，所以我们用这接口的实现类，比如（ApplicationContext ctx = new ClassPathXmlApplicationContext("com/yf/context/beans.xml")）

IOC的生命周期可以总结为：oc容器创建（其实就是类实例化，但是还没有属性赋值等操作）-------->加载所有bean的定义信息-------->创建bean的实例--------->属性赋值（调用bean的setxxx方法）---------->假如bean实现了xxxAware接口，就执行setXXX方法---------->执行初始化方法（相当于xml<bean init-method="初始化方法">）----------->ioc容器创建完成，执行一些逻辑代码------------->Web应用关闭，ioc容器销毁。



Spring IOC ：

1. Bean定义的定位,Bean 可能定义在XML中，或者一个注解，或者其他形式。这些都被用Resource来定位, 读取Resource获取BeanDefinition 注册到 Bean定义注册表中。
2. 第一次向容器getBean操作会触发Bean的创建过程,实列化一个Bean时 ,根据BeanDefinition中类信息等实列化Bean.
3. 将实列化的Bean放到单列Bean缓存内。
4. 此后再次获取向容器getBean就会从缓存中获取。

主要为refresh方法，在AbstractAplicationContext中：

```java
  public void refresh() throws BeansException, IllegalStateException {
        synchronized(this.startupShutdownMonitor) {
            this.prepareRefresh();
            ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
            this.prepareBeanFactory(beanFactory);

            try {
                this.postProcessBeanFactory(beanFactory);
                this.invokeBeanFactoryPostProcessors(beanFactory);
                this.registerBeanPostProcessors(beanFactory);
                this.initMessageSource();
                this.initApplicationEventMulticaster();
                this.onRefresh();
                this.registerListeners();
                this.finishBeanFactoryInitialization(beanFactory);
                this.finishRefresh();
            } catch (BeansException var9) {
                if (this.logger.isWarnEnabled()) {
                    this.logger.warn("Exception encountered during context initialization - cancelling refresh attempt: " + var9);
                }

                this.destroyBeans();
                this.cancelRefresh(var9);
                throw var9;
            } finally {
                this.resetCommonCaches();
            }

        }
    }
```

ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory(); 中进入代码后，有一个refreshBeanFactory方法，该方法判断了是否存在基础的BeanFactory容器，有的话就销毁，接着用 createBeanFactory() 方法创建了一个 DefaultListableBeanFactory。ApplicationContext 是在基础 BeanFactory 上添加了高级容器特征的 IoC 容器，而且大多数情况下是使用 DefaultListableBeanFactory 这个具有基础容器功能的 BeanFactory。接着就是 loadBeanDefinitions 调用的地方，首先得到 BeanDefinition 的 Resource 定位。





### IOC

IoC容器的初始化过程可以分为三步：

1. Resource定位（Bean的定义文件定位）
2. 将Resource定位好的资源载入到BeanDefinition
3. 将BeanDefiniton注册到容器中

一、Resource定位：

 Resource是Spring中用于封装I/O的接口，在创建spring容器时，通常要访问XML配置文件，除此之外还可以通过访问文件类型、二进制流等方式访问资源，还有当需要网络上的资源时可以通过访问URL，Spring把这些文件统称为Resource。 其次Spring 提供了ResourceLoader接口用于实现不同的Resource加载策略，该接口的实例对可以获取一个resource对象，ResourceLoader定义了

```java
Resource getResource(String location); //通过提供的资源location参数获取Resource实例
ClassLoader getClassLoader(); // 获取ClassLoader,通过ClassLoader可将资源载入JVM
```

ApplicationContext的所有实现类都实现RecourceLoader接口，因此可以直接调用getResource（参数）获取Resoure对象**。**不同的ApplicatonContext实现类使用getResource方法取得的资源类型不同

二、通过返回的resource对象，进行BeanDefinition的载入

BeanDefinition相当于一个数据结构，这个数据结构的生成过程根据定位的resource资源对象中的bean而来，这些bean在springIoc容器内部成 了BeanDefinition这样的数据结构，IOC容器对bean的管理和依赖注入都是通过BeanDefinition来进行的。

在Spring中配置文件主要格式是XML，对于用来读取XML型资源文件来进行初始化的IoC 容器而言，该类容器会使用到AbstractXmlApplicationContext类，该类定义了一个名为loadBeanDefinitions(DefaultListableBeanFactory beanFactory) 的方法用于获取BeanDefinition，该方法new一个与容器对应的BeanDefinitionReader型实例对象，然后将生成的BeanDefintionReader实例作为参数传入loadBeanDefintions(XmlBeanDefinitionReader)，继续往下执行载入BeanDefintion的过程。将所有定位的resource的资源位置（用户定义）以及本地所有配置文件的位置即容器本身资源全部加载到reader中，并调用reader.loadBeanDefinitions方法。  处理的过程主要可以为，先把resource包装为EncodeResource类型，之后读取resource对象得到XML的文件流，从资源中加载Docunment对象，最后将document文件的bean封装成BeanDefinition,并注册到容器。

三、将ReanDefinition注册到容器中

最终Bean配置会被解析成BeanDefinition并与beanName,Alias一同封装到**BeanDefinitionHolder类**中， 之后beanFactory.registerBeanDefinition(beanName, bdHolder.getBeanDefinition())，注册到**DefaultListableBeanFactory**.beanDefinitionMap中。之后客户端如果要获取Bean对象，Spring容器会根据注册的BeanDefinition信息进行实例化。（**beanDefinitionMap是个ConcurrentHashMap类型数据，用于存放beanDefinition,它的key值是beanName**）



#### aop

方面（Aspect）：一个关注点的模块化，这个关注点实现可能另外横切多个对象。事务管理是一个很好的横切关注点例子。方面用Spring的Advisor或拦截器实现。

连接点（Joinpoint）: 程序执行过程中明确的点，如方法的调用或特定的异常被抛出。

通知（Advice）: 在特定的连接点，AOP框架执行的动作。各种类型的通知包括“around”、“before”和“throws”通知。通知类型将在下面讨论。许多AOP框架包括Spring都是以拦截器做通知模型，维护一个“围绕”连接点的拦截器链。Spring中定义了四个advice: BeforeAdvice, AfterAdvice, ThrowAdvice和DynamicIntroductionAdvice

切入点（Pointcut）: 指定一个通知将被引发的一系列连接点的集合。AOP框架必须允许开发者指定切入点：例如，使用正则表达式。 Spring定义了Pointcut接口，用来组合MethodMatcher和ClassFilter，可以通过名字很清楚的理解， MethodMatcher是用来检查目标类的方法是否可以被应用此通知，而ClassFilter是用来检查Pointcut是否应该应用到目标类上

目标对象（Target Object）: 包含连接点的对象。也被称作被通知或被代理对象。

AOP代理（AOP Proxy）: AOP框架创建的对象，包含通知。 在Spring中，AOP代理可以是JDK动态代理或者CGLIB代理。

aop可以分为静态代理和动态代理

静态代理是指使用aop框架提供的命令进行编译，从而在编译阶段就可以生成aop代理类，因此也称为编译时增强，动态代理则是在运行时借助JDK动态代理、CGLIB等在内存中‘’临时‘’生成aop动态代理类，因此也被称为运行时增强

 JDK动态代理：默认地，如果使用接口的，用 JDK 提供的动态代理实现，如果没有接口，使用 CGLIB 实现

- Spring 1.2 **基于接口的配置**：最早的 Spring AOP 是完全基于几个接口的
- Spring 2.0 **schema-based 配置**：Spring 2.0 以后使用 XML 的方式来配置，使用 命名空间 `<aop />`
- Spring 2.0 **@AspectJ 配置**：使用注解的方式来配置，这种方式感觉是最方便的，还有，这里虽然叫做 `@AspectJ`，但是这个和 AspectJ 其实没啥关系。

用Advisor 表示 用于拦截的类，advice 是指用来切入的方法。最后需要定义需要代理的类

```xml
<aop:config>
    <aop:aspect ref="logArgsAspect">  //，一个bean 该bean中写处理代码
        <aop:pointcut id="internalPointcut" //具体实现的方法织入到合适的pointcut
                expression="com.javadoop.SystemArchitecture.businessService()" />
    </aop:aspect>
</aop:config>
// pointcut 就是配置我们需要拦截哪些方法，接下来，我们要配置需要对这些被拦截的方法做什么，也就是前面介绍的 Advice
```

通过配置Spring的中<<aop:config>>标签来显示的指定使用动态代理机制 proxy-target-class=true表示使用CGLib代理，如果为false就是默认使用JDK动态代理

**JDK动态代理**

1. 通过实现 InvocationHandler 接口创建自己的调用处理器；
2. 通过为 Proxy 类指定 ClassLoader 对象和一组 interface 来创建动态代理类；
3. 通过反射机制获得动态代理类的构造函数，其唯一参数类型是调用处理器接口类型；
4. 通过构造函数创建动态代理类实例，构造时调用处理器对象作为参数被传入。 

**GCLIB代理** 

1. cglib（Code Generation Library）是一个强大的,高性能,高质量的Code生成类库。它可以在运行期扩展Java类与实现Java接口。
2. cglib封装了asm，可以在运行期动态生成新的class。

区别： 

1. JDK的动态代理必须基于接口，CGLIB没有这个要求，如果目标对象实现了接口，默认情况下采用JDK动态代理，可以强制使用CGLIB，如果没实现接口，必须采用CGLIB
2. java动态代理是利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。而cglib动态代理是利用asm开源包，对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。
3. JDK动态代理只能对实现了接口的类生成代理，而不能针对类 
4. CGLIB是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法因为是继承，所以该类或方法最好不要声明成final

JDK：动态代理类JDKProxy，实现InvocationHandler接口，并且实现接口中的invoke方法。当客户端调用代理对象的业务方法时，代理对象执行invoke方法，invoke方法把调用委派给targetObject，相当于调用目标对象的方法，在invoke方法委派前判断权限，实现方法的拦截

```java
 public Object createProxyInstance(Object targetObject) {
        this.targetObject = targetObject;
        return Proxy.newProxyInstance(this.targetObject.getClass().getClassLoader(),
                this.targetObject.getClass().getInterfaces(), this);
    }

```

CGLIB:实现了创建子类的方法与代理的方法。getProxy(SuperClass.class)方法通过入参即父类的字节码，扩展父类的class来创建代理对象。intercept()方法拦截所有目标类方法的调用，obj表示目标类的实例，method为目标类方法的反射对象，args为方法的动态入参，methodProxy为代理类实例。method.invoke(targetObject, args)通过代理类调用父类中的方法。

```java
 public Object createProxyObject(Object obj) {
        this.targetObject = obj;
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(obj.getClass());
        //回调方法的参数为代理类对象CglibProxy，最后增强目标类调用的是代理类对象CglibProxy中的intercept方法 
        enhancer.setCallback(this);
        //增强后的目标类
        Object proxyObj = enhancer.create();
        // 返回代理对象
        return proxyObj;
    }

```







**select的几大缺点：**

**（1）每次调用select，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大**

**（2）同时每次调用select都需要在内核遍历传递进来的所有fd，这个开销在fd很多时也很大**

**（3）select支持的文件描述符数量太小了，默认是1024**

epoll的提升：

1. 本身没有最大并发连接的限制，仅受系统中进程能打开的最大文件数目限制；
2. 效率提升：只有活跃的socket才会主动的去调用callback函数；
3. 省去不必要的内存拷贝：epoll通过内核与用户空间mmap同一块内存实现。
4. epoll提供了三个函数，epoll_create,epoll_ctl和epoll_wait，epoll_create是创建一个epoll句柄；epoll_ctl是注册要监听的事件类型；epoll_wait则是等待事件的产生， 每一次注册新的事件。都会把所有的fd拷贝进内核，而不是在epoll_wait的时候重复拷贝，保证了每一个fd在整个过程只会拷贝一次。
5. epoll的解决方案不像select或poll一样每次都把current轮流加入fd对应的设备等待队列中，而只在epoll_ctl时把current挂一遍（这一遍必不可少）并为每个fd指定一个回调函数，当设备就绪，唤醒等待队列上的等待者时，就会调用这个回调函数，而这个回调函数会把就绪的fd加入一个就绪链表）。epoll_wait的工作实际上就是在这个就绪链表中查看有没有就绪的fd（利用schedule_timeout()实现睡一会，判断一会的效果，和select实现中的第7步是类似的）
6. epoll所支持的FD上限是最大可以打开文件的数目，这个数字一般远大于2048

总结：

1）select，poll实现需要自己不断轮询所有fd集合，直到设备就绪，期间可能要睡眠和唤醒多次交替。而epoll其实也需要调用epoll_wait不断轮询就绪链表，期间也可能多次睡眠和唤醒交替，但是它是设备就绪时，调用回调函数，把就绪fd放入就绪链表中，并唤醒在epoll_wait中进入睡眠的进程。虽然都要睡眠和交替，但是select和poll在“醒着”的时候要遍历整个fd集合，而epoll在“醒着”的时候只要判断一下就绪链表是否为空就行了，这节省了大量的CPU时间。这就是回调机制带来的性能提升。

（2）select，poll每次调用都要把fd集合从用户态往内核态拷贝一次，并且要把current往设备等待队列中挂一次，而epoll只要一次拷贝，而且把current往等待队列上挂也只挂一次（在epoll_wait的开始，注意这里的等待队列并不是设备等待队列，只是一个epoll内部定义的等待队列）。这也能节省不少的开销。