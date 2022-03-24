
1.SpringAOP的底层原理

---

2.SpringBean的生命周期

---

Bean对象的生命周期是由IOC进行管理的，在Spring中的基础容器是ApplicationContext,他是顶层容器接口BeanFactory的实现类，总的来说，Bean的生命周期包括了**创建时准备，创建实例，依赖注入，容器缓存，销毁实例**几个流程，基本思路是，在Bean出现之前，先准备操作Bean的BeanFactory，然后操作完Bean，将Bean交给BeanFactory管理，总的来说，归位以下几个阶段

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

接下来，工厂会对已注册的**Bean实施实例化**策略，现有的Bean实例化策略是在AbstractAutoWireCapleBleBeanFactory使用BeanClass.newInstance()来实例化，仅仅用于Bean的无参构造，往下走就是为**Bean填充属性，**在BeanDefinition中增加和Bean属性对应的PropertyValues，实施Bean之后，为Bean填充属性
