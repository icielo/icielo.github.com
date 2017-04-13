---
layout: post
title: "spring profile"
description: "spring boot 根据不同的环境参数启动"
tags: [spring profile]
categories: [spring]
published: true
---

## spring boot 根据不同的环境参数启动。

### applicationContext配置，其中app.properties是共同配置，app.*.properties是差异性配置

```xml
<!-- 数据源 -->
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
	<property name="driverClassName" value="${jdbc.driverClassName}" />
	<property name="url" value="${jdbc.url}" />
	<property name="username" value="${jdbc.username}" />
	<property name="password" value="${jdbc.password}" />
	<!-- 配置连接池的初始值 -->
	<property name="initialSize" value="1" />
	<!-- 连接池的最大值 -->
	<!-- <property name="maxActive" value="500"/> -->
	<!-- 最大空闲时，当经过一个高峰之后，连接池可以将一些用不到的连接释放，一直减少到maxIdle为止 -->
	<!-- <property name="maxIdle" value="2"/> -->
	<property name="minIdle" value="1" />
	<property name="maxActive" value="100" />
	<property name="maxIdle" value="20" />
	<property name="maxWait" value="1000" />
</bean>

<!-- 开发环境配置文件 -->  
<beans profile="development">
  <context:property-placeholder location="classpath:app.properties,classpath:env/app.development.properties"/>  
</beans>  

<!-- 测试环境配置文件 -->  
<beans profile="test">
  <context:property-placeholder location="classpath:app.properties,classpath:env/app.test.properties"/>
</beans>  

<!-- 预生产环境配置文件 -->  
<beans profile="functional">  
  <context:property-placeholder location="classpath:app.properties,classpath:env/app.functional.properties"/>  
</beans>  

<!-- 生产环境配置文件 -->  
<beans profile="production">  
  <context:property-placeholder location="classpath:app.properties,classpath:env/app.production.properties"/>  
</beans>  

<!-- 默认配置 -->
<beans profile="default">  
  <context:property-placeholder ignore-resource-not-found="true" location="classpath:app.properties"/>
</beans> 
```
### web系统启动监听器,获取启动参数，初始化到系统配置类中

```java
public class CustomContextLoaderListener extends ContextLoaderListener {
	
	protected Logger logger = LoggerFactory.getLogger(getClass());

	@Override
	public void contextInitialized(ServletContextEvent event) {
		super.contextInitialized(event);
		//启动参数
		Global.ENV = event.getServletContext().getInitParameter("spring.profiles.active");
		if(StringUtils.isBlank(Global.ENV)){
			Global.ENV	= System.getProperty("spring.profiles.active","development");
		}
		System.setProperty("spring.profiles.active",Global.ENV);
		logger.info("系统启动环境为：" + Global.ENV);
		Global.init();
	}
	
}
```

### 系统配置类

```java
public class Global {
  
  	/** 环境参数 */
	public static String ENV;
  
  	/**
	 * 环境差异化属性加载
	 */
	private static PropertiesLoader envLoader = null;
	
	/**
	 * 属性文件加载对象
	 */
	private static PropertiesLoader loader = null;
  
  	/**
	 * 保存全局属性值
	 */
	private static Map<String, String> map = Maps.newHashMap();
  	
	/**
	 * 初始化全局参数
	 */
	public static void init() {
		loader = new PropertiesLoader("app.properties");
		String fileName = "env" + File.separator +"app." + Global.ENV +".properties";
		envLoader = new PropertiesLoader(fileName);
	}
  	
  	/**
	 * 获取配置
	 */
	public static String getConfig(String key) {
		String value = map.get(key);
		if (org.apache.commons.lang3.StringUtils.isBlank(value)){
			value = envLoader.getProperty(key);
			if(org.apache.commons.lang3.StringUtils.isBlank(value)){
				value = loader.getProperty(key);	
			}
			map.put(key, value != null ? value : StringUtils.EMPTY);
		}
		return value;
	}
}
```

### 配置启动参数方式

1. 工程web.xml激活

   ```xml
   <!--  启动环境参数 -->
   <context-param>  
   	<param-name>spring.profiles.active</param-name>  
   	<param-value>development</param-value>  
   </context-param>
   ```

2. Tomcat激活 conf/web.xml

   ```xml
   <context-param>  
   	<param-name>spring.profiles.active</param-name>  
   	<param-value>development</param-value>  
   </context-param>
   ```

3. JVM激活，配置Tomcat的JVM启动参数

   ```
   -Dspring.profiles.active="development"
   ```

   ### 优先级 工程web.xml > tomcat的web.xml > JVM。默认启动方式development

