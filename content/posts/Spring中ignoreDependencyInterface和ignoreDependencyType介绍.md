---
title: Spring中ignoreDependencyInterface和ignoreDependencyType介绍
date: 2021-07-30 15:53:08
categories: Spring
tags:
- Spring

---

ignoreDependencyInterface和ignoreDependencyType的使用以及作用的介绍。

<!--more-->

# ignoreDependencyInterface

Spring中可以使用自动装配来设置Bean的依赖，比如如下代码：

```java
public class UserServiceImpl implements UserService {

    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public String getUserName(Long userId) {
        return userDao.queryUserName(userId);
    }
}
```

这里UserService需要注入依赖UserDao，xml中的配置如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans-3.0.xsd" default-autowire="byType">

    <bean id="userDao" class="me.cxis.spring.autowire.xml.dao.impl.UserDaoImpl"/>
    <bean id="userService" class="me.cxis.spring.autowire.xml.service.impl.UserServiceImpl"/>

</beans>
```

配置中使用了default-autowire="byType"或者byName，可以让userDao自动注入到userService中去，这里使用的是setter方法进行的注入。在Spring中还有一种Aware的使用方式，常用示例如下：

```java
public class ApplicationContextUtilXml implements ApplicationContextAware {

    private static ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    public static Object getBean(String name) {
        return applicationContext.getBean(name);
    }
}
```

xml配置如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans-3.0.xsd" default-autowire="byType">

    <bean id="userDao" class="me.cxis.spring.autowire.xml.dao.impl.UserDaoImpl"/>
    <bean id="userService" class="me.cxis.spring.autowire.xml.service.impl.UserServiceImpl"/>

    <bean id="applicationContextUtilXml" class="me.cxis.spring.utils.ApplicationContextUtilXml"/>

</beans>
```

这种方式中ApplicationContextUtilXml中的applicationContext并不使用上面的自动注入通过setter方法进行注入，为了不让Spring容器通过自动注入方式注入类似这种Aware接口的set方法中的Bean，Spring通过ignoreDependencyInterface方法来将这些接口（Aware）进行忽略，遇到需要自动装配的时候如果发现是需要忽略的接口（Aware的setXXX方法），则不进行set方法的自动注入，而是使用容器的另外的方法进行注入这些setXXX（实际是在initializeBean的时候直接调用的各个Aware的setXXX方法进行的设置）。

# ignoreDependencyType

这个ignoreDependencyType设置了之后，设置的Bean将没办法被注入到其他的Bean中去，示例代码如下：

```java
public class IgnoreDependencyType implements BeanFactoryPostProcessor {

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
        // 加上这个的话，UserDao就不会被注入到其他的Bean中
        configurableListableBeanFactory.ignoreDependencyType(UserDao.class);
    }
}
```

如果加上了这个，UserService中依赖的UserDao将不会被自动注入进去。

