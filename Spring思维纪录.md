
1.SpringAOP的底层原理

---

2.SpringBean的生命周期

---

Bean对象的生命周期是由IOC进行管理的，在Spring中的基础容器是ApplicationContext,他是顶层容器接口BeanFactory的实现类，总的来说，Bean的生命周期包括了**创建时准备，创建实例，依赖注入，容器缓存，销毁实例**几个流程，基本思路是，在Bean出现之前，先准备操作Bean的BeanFactory，然后操作完Bean，将Bean交给BeanFactory管理，总的来说，归为以下几个阶段

- Bean创建前的准备

Bean容器在配置文件中找到SpringBean的定义以及相关配置，比如初始化方法和销毁方法，接下来会实例化回调相关的后置处理器比如BeanFactoryPostProcessor,BeanPostProcessor等，一般来说Bean容器对Bean的定义要经过以下几个步骤,如下代码

```java
public class BeanDefinitionAndBeanDefinitionRegistryTest {
	@Test
	public void testBeanFactory() throws Exception {
		// 初始化Bean工厂实例
		DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
		// BeanDefinition是用于定义Bean信息的类，包括Bean的class类型，构造参数，属性等信息，每个Bean都会关联一个BeanDefinetion例
		BeanDefinition beanDefinition = new BeanDefinition(HelloService.class);
		// BeanDefinitionRegistry,BeanDefinition注册表接口，定义注册BeanDefinition的方法
		beanFactory.registerBeanDefinition("helloService", beanDefinition);
		HelloService helloService = (HelloService) beanFactory.getBean("helloService");
		helloService.sayHello();
	}
}
```

接下来，工厂会对已注册的**Bean实施实例化**策略，现有的Bean实例化策略是在AbstractAutoWireCapleBleBeanFactory使用BeanClass.newInstance()来实例化，仅仅用于Bean的无参构造，往下走就是为**Bean填充属性**， 在BeanDefinition中增加和Bean属性对应的PropertyValues，实施Bean之后，为Bean填充属性，在此期间可能会增加BeanReference类来包装一个Bean对另一个Bean的引用，在这里可能会出现循环引用的，在这里记录一下

> 所谓的Bean循环引用，其实就是Spring在初始化过程中，会按照BeanDefinitionNames的顺序，也就是Bean的注册顺序来依次初始化所有的Bean，对所有的Bean会调用一次getBean，然后由BeanFactory进行初始化

例如下面这段代码
```java
public class CircularReferenceWithoutProxyBeanTest {

	@Test
	public void testCircularReference() throws Exception {
		ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:circular-reference-without-proxy-bean.xml");
		A a = applicationContext.getBean("a", A.class);
		B b = applicationContext.getBean("b", B.class);
		assertThat(a.getB() == b).isTrue();
	}
}
```
我们从XML文件中按照顺序读取出了A和B对象，但是在XMl文件中显示两个对象之间是互相引用的
```xml
    <bean id="b" class="org.springframework.test.bean.B">
        <property name="a" ref="a"/>
    </bean>

    <!-- a被代理 -->
    <bean id="a" class="org.springframework.test.bean.A">
        <property name="b" ref="b"/>
    </bean>
```
我们都知道初始化Bean有两个关键流程，一个是创建Bean对象，一个是填充Bean的属性值，如果出现了上面这种情况，可能ABean在填充过程中发现了B的引用，要先去对BBean进行初始化操作，但是BBean在初始化的时候也发现了A的引用，需要回去创建A，所以就会出现死循环的情况，先来看下Bean的整个初始化流程图,这些全部都是在doCreateBean这个函数中完成的

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a4df85e9968403bbe5c9200fe7e1bd4~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

1.Bean的实例化

Bean的实例化实在AbstractAutowireCapableBeanFactory中进行的，这个类是实现了AbstractCapableBeanFactory接口，而AbstractCapableBeanFactory接口则是实现了BeanFactory接口

![](https://github.com/DerekYRC/mini-spring/raw/main/assets/bean-definition-and-bean-definition-registry.png)

Bean的实例化会调用**AbstractAutowireCapableFactory类的createBeanInstance**方法实例化Bean,开始时，会去判断Bean是否需要被代理，有的话可以直接返回对象，这一步其实就是**实力化的前置处理操作**，经过前置处理后返回的结果如果不为空，那么就会直接略过后续的Bean的创建从而直接返回结果

```java
	@Override
	protected Object createBean(String beanName, BeanDefinition beanDefinition) throws BeansException {
		//如果bean需要代理，则直接返回代理对象
		Object bean = resolveBeforeInstantiation(beanName, beanDefinition);
		if (bean != null) {
			return bean;
		}

		return doCreateBean(beanName, beanDefinition);
	}
```
