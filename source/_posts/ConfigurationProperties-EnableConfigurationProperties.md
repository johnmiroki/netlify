title: '@ConfigurationProperties & @EnableConfigurationProperties'
author: John Miroki
tags:
  - TIL
  - SpringBoot
categories: []
date: 2018-07-12 00:12:00
---
Simply put, @EnableConfigurationProperties is acted like a general switch, enabling or disabling (by absense) Spring Boot's functionality to bind a certain properties from the environment to a pojo class, and @ConfigurationProperties is used on the said pojo, as the receiver of environmental properties.

These two annotations should be both used at the same time in combinations in a project, as follows:

1. 
on the app's configuration class:
```java
@SpringBootApplication
@EnableConfigurationProperties
public class SpringBootWebApplication {...}
```
and one the pojo
```
@Component
@ConfigurationProperties
public class TestConfig {}
```

In this case, @EnableConfigurationProperties enables the functionality app-wise, and the config pojo registers itself with the help of @Component.

2.
on the app's configuration class:
```java
@SpringBootApplication
@EnableConfigurationProperties(TestConfig.class)
public class SpringBootWebApplication {...}
```
and one the pojo
```
@ConfigurationProperties
public class TestConfig {}
```

In this case, @EnableConfigurationProperties not only enables the functionality, but also selects a config pojo to be used in this project. The config pojo (TestConfig) doesn't have to register itself through @Component.
