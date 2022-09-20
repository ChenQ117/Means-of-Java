# Spring—面试

## refresh

### 1.prepareRefresh

- 创建和准备了Environment对象
- Environment的作用之一：为后续@Value值注入时提供键值
- Environment中含有：
  - systemProperties
  - systemEnvironment
  - 自定义PropertySource

### 2.obtainFreshBeanFactory

- 获取或创建BeanFactory
- BeanFactory的作用：负责bean的创建，依赖注入和初始化
- BeanDefinition作为bean的设计蓝图，规定了bean的特征，如单例多例、依赖关系、初始销毁方法等
- BeanDefinition的来源有多种多样，可以通过xml获得（XMLBeanDefinitionReader）、通过配置类获得、通过组件扫描获取，也可以编程添加
- Environment和BeanFactory是组合关系

### 3.prepareBeanFactory

- 完善BeanFactory
- StandardBeanExpressionResolver来解析SpEL
- ResourceEditorRegistrar会注释类型转换器，并应用ApplicationContext提供的Environment完成${}解析
- 特殊bean指beanFactory以及ApplicationContext，通过registerResolvableDependency来注册它们
- ApplicationContextAwareProcessor用来解析Aware接口

### 4.postProcessBeanFactory

- 空实现，留给之类扩展
- 一般web环境ApplicationContext都要利用它注册新的Scope，完善Web下的BeanFactory
- 体现的是模板方法设计模式

### 5.invokeBeanFactoryPostProcessors

- beanFactory后处理器，充当beanFactory的扩展点，可以用来补充或者修改BeanDefinition
- ConfigurationClassPostProcessor-解析@Configuration、@Bean、@Import、@PropertySource等
- PropertySourcesPlaceHolderConfigurer -替换BeanDefinition中的${}