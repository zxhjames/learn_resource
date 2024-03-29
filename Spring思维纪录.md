
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

整个流程可以如图下表示出来
![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/2/15/1704860a4de235aa~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)
<!-- ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a4df85e9968403bbe5c9200fe7e1bd4~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp) -->

从头
1.Bean的实例化

Bean的实例化实在AbstractAutowireCapableBeanFactory中进行的，这个类是实现了AbstractCapableBeanFactory接口，而AbstractCapableBeanFactory接口则是实现了BeanFactory接口

![](https://github.com/DerekYRC/mini-spring/raw/main/assets/bean-definition-and-bean-definition-registry.png)

Bean的实例化会调用**AbstractAutowireCapableFactory类的createBeanInstance**方法实例化Bean,开始时，会去判断Bean是否需要被代理，有的话可以直接返回对象，这一步其实就是**实力化的前置处理操作**，经过前置处理后返回的结果如果不为空，那么就会直接略过后续的Bean的创建从而直接返回结果,所谓的实例化前置，就是在对象进行实例化之前对Bean对象的class信息进行扩展或者修改，以达到我们想要的功能，它的底层是动态代理AOP实现的，并且是Bean周期中最先执行的方法，这个过程我们要知道，是用反射技术创建的，只是相当于new出来了一个对象而已，但是这个时候的只是将对象进行实例化了，对象内的属性值还没有设置，如果返回的对象为空，则会开始真正创建Bean的过程

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
我们进入到doCreateBean方法中，可以看到以下的代码

```java
protected Object doCreateBean(String beanName, BeanDefinition beanDefinition) {
		Object bean;
		try {
			bean = createBeanInstance(beanDefinition);

			//为解决循环依赖问题，将实例化后的bean放进缓存中提前暴露
			if (beanDefinition.isSingleton()) {
				Object finalBean = bean;
				addSingletonFactory(beanName, new ObjectFactory<Object>() {
					@Override
					public Object getObject() throws BeansException {
						return getEarlyBeanReference(beanName, beanDefinition, finalBean);
					}
				});
			}

			//实例化bean之后执行
			boolean continueWithPropertyPopulation = applyBeanPostProcessorsAfterInstantiation(beanName, bean);
			if (!continueWithPropertyPopulation) {
				return bean;
			}
			//在设置bean属性之前，允许BeanPostProcessor修改属性值
			applyBeanPostProcessorsBeforeApplyingPropertyValues(beanName, bean, beanDefinition);
			//为bean填充属性
			applyPropertyValues(beanName, bean, beanDefinition);
			//执行bean的初始化方法和BeanPostProcessor的前置和后置处理方法
			bean = initializeBean(beanName, bean, beanDefinition);
		} catch (Exception e) {
			throw new BeansException("Instantiation of bean failed", e);
		}

		//注册有销毁方法的bean
		registerDisposableBeanIfNecessary(beanName, bean, beanDefinition);

		Object exposedObject = bean;
		if (beanDefinition.isSingleton()) {
			//如果有代理对象，此处获取代理对象
			exposedObject = getSingleton(beanName);
			addSingleton(beanName, exposedObject);
		}
		return exposedObject;
	}
```

BeanPostProcessor的两个方法分别在Bean执行初始化方法(后面实现)之前和之后执行，是Spring提供的容器扩展机制，不同于BeanFactoryPostProcessor的是，BeanPostProcessor在Bean实例化后修改Bean或者替换Bean，是实现AOP的关键

```java
public interface BeanPostProcessor {

	/**
	 * 在bean执行初始化方法之前执行此方法
	 *
	 * @param bean
	 * @param beanName
	 * @return
	 * @throws BeansException
	 */
	Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;

	/**
	 * 在bean执行初始化方法之后执行此方法
	 *
	 * @param bean
	 * @param beanName
	 * @return
	 * @throws BeansException
	 */
	Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;
}

```

相比于BeanFactory,ApplicationContext是Spring中较BeanFactory更为先进的IOC容器，ApplicationContext除了拥有BeanFactory的所有功能外，还支持特殊类型bean如上一节中的BeanFactoryPostProcessor和BeanPostProcessor的自动识别、资源加载、容器事件和监听器、国际化支持、单例bean自动初始化等。注意这里的一大特点就是自动识别BeanFactoryPostProcessor和BeanPostProcessor,BeanFactory是Spring的基础设施，面向本身，ApplicationContext是面向使用者的

2. Bean的初始化方法和销毁方法

在Spring中，定义Bean的初始化和销毁方法有三种
* 在xml中自定义init-method和destroy-method,这种方式需要在BeanDefinition中增加属性initMethodName和destroyMethodName.初始化方法在AbstractAutowireCapableBeanFactory执行，
* 继承自initializingBean和DisposalbleBean
* 在方法上加注解PostConstruct和PreDestroy

3.循环依赖问题以及三级缓存问题
在Spring体系中，如果出现A依赖于B，B依赖于C，C依赖于A，就会出现闭环，从而导致循环依赖问题，Spring解决循环依赖的核心就是提前暴露对象，在这里，一级缓存用于存放完整的Bean，二级缓存用于存放提前暴露的Bean，Bean是不完整的，未完成属性注入和执行init方法，三级缓存，存放的是Bean工厂，主要是生产Bean，存放到二级缓存中

所有被Spring管理的Bean，最终都会放在singletonObjects中，这里存放的Bean是经历了所有生命周期的(除了销毁)，完整的，可以被用户使用的，earlySingletonObject

	
	
