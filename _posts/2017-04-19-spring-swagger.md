---
layout: post
title: "spring swagger"
description: "spring quartz定时任务分布式集群"
tags: [spring swagger]
categories: [spring]
published: true
---

## 在spring MVC的RESTful接口上添加swagger支持。swagger可以自动扫描对外提供的接口服务，生成API信息。通过swagger UI提供可视化界面，展示可用API接口、提交方式、请求参数及其模板等，并提供实时调用。

### MAVEN依赖包

```xml
<!-- swagger-springmvc -->
<dependency>
	<groupId>com.mangofactory</groupId>
	<artifactId>swagger-springmvc</artifactId>
	<version>1.0.2</version>
</dependency>
<dependency>
    <groupId>com.mangofactory</groupId>
    <artifactId>swagger-models</artifactId>
    <version>1.0.2</version>
</dependency>
<dependency>
    <groupId>com.wordnik</groupId>
    <artifactId>swagger-annotations</artifactId>
    <version>1.3.11</version>
</dependency>
<!-- swagger-springmvc dependencies -->

<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>15.0</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.4.4</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.4.4</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.4.4</version>
</dependency>
<dependency>
    <groupId>com.fasterxml</groupId>
    <artifactId>classmate</artifactId>
    <version>1.1.0</version>
</dependency>
```

### 新增SwaggerConfig 配置类

```java
/**
 * lincl
 * 2016年8月26日 上午11:26:14
 * 
 */
package com.lezic.tiana.api.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.mangofactory.swagger.configuration.SpringSwaggerConfig;
import com.mangofactory.swagger.models.dto.ApiInfo;
import com.mangofactory.swagger.plugin.EnableSwagger;
import com.mangofactory.swagger.plugin.SwaggerSpringMvcPlugin;

/**
 * @author lincl
 * 
 */
@Configuration
@EnableSwagger
public class SwaggerConfig {

    @Autowired
    private SpringSwaggerConfig springSwaggerConfig;

    @Bean
    public SwaggerSpringMvcPlugin customImplementation() {
        return new SwaggerSpringMvcPlugin(this.springSwaggerConfig).apiInfo(apiInfo()).includePatterns(".*?");
    }

    private ApiInfo apiInfo() {
        ApiInfo apiInfo = new ApiInfo("My Apps API Title", "My Apps API Description", "My Apps API terms of service",
                "My Apps API Contact Email", "My Apps API Licence Type", "My Apps API License URL");
        return apiInfo;
    }
}
```

### spring bean注入以及swagger UI映射路径

```xml
<!-- swagger -->
<bean id="springSwaggerConfig" class="com.mangofactory.swagger.configuration.SpringSwaggerConfig" />
<mvc:resources mapping="/swagger/**" location="/WEB-INF/swagger/" />
```

### 引入swagger-ui

- https://github.com/swagger-api/swagger-ui/releases
- 选择zip包打包，如 [Source code (zip)](https://github.com/swagger-api/swagger-ui/archive/v2.2.2.zip)
- 解压缩，将dist目录内容拷贝到 /webapp/WEB-INF/swagger目录
- 中文支持.。修改swagger目录下的index.html,新增语言支持
```javascript
<!-- Some basic translations -->
  <script src='lang/translator.js' type='text/javascript'></script>
  <script src='lang/ru.js' type='text/javascript'></script>
  <script src='lang/zh-cn.js' type='text/javascript'></script>
```
- 修改默认api路径
```javascript
$(function () {
      var url = window.location.search.match(/url=([^&]+)/);
      if (url && url.length > 1) {
        url = decodeURIComponent(url[1]);
      } else {
        //修改默认api路径
        url = window.location.href.replace(/swagger\/index.html.*/g,"api-docs");
      }
    ......
});
```

### 新增 学生类 Student

```java
/**
 * lincl
 * 2016年8月26日 下午3:24:52
 * 
 */
package com.lezic.tiana.api.demo;

import java.io.Serializable;

/**
 * 学生
 * 
 * @author lincl
 * 
 */
public class Student implements Serializable {

    private static final long serialVersionUID = 1L;

    /** 主键 */
    private Integer id;

    /** 姓名 */
    private String name;

    /** 性别 */
    private String sex;

    /** 年龄 */
    private Integer age;

    /** 年级 */
    private String grade;
    
    
    public Student() {
        super();
    }

    public Student(int id, String name, String sex, int age, String grade) {
        super();
        this.id = id;
        this.name = name;
        this.sex = sex;
        this.age = age;
        this.grade = grade;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getGrade() {
        return grade;
    }

    public void setGrade(String grade) {
        this.grade = grade;

    }

}
```

### 新增学生controller StudentController

```java
/**
 * lincl
 * 2016年8月26日 下午3:20:37
 * 
 */
package com.lezic.tiana.api.demo;

import java.util.ArrayList;
import java.util.List;

import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import com.lezic.tiana.constant.AjaxResult;
import com.lezic.tiana.constant.StatusCode;
import com.lezic.tiana.util.DataUtil;
import com.lezic.tiana.web.BaseController;
import com.wordnik.swagger.annotations.ApiOperation;

/**
 * 这是关于学生的例子
 * 
 * @author lincl
 * 
 */
@RestController
@RequestMapping("/students")
public class StudentController extends BaseController {

	private static List<Student> list = new ArrayList<Student>();

	private static int id = 4;

	static {
		list.add(new Student(1, "张三", "男", 16, "高二"));
		list.add(new Student(2, "小明", "男", 15, "高一"));
		list.add(new Student(3, "小红", "女", 14, "高一"));
	}

	/**
	 * 根据学生属性进行搜索
	 * 
	 * @param req
	 * @return
	 * @author lincl
	 * @date 2016年8月26日 下午6:25:36
	 */
	@RequestMapping(value = "/list", method = RequestMethod.POST)
	@ApiOperation(value = "根据学生属性进行搜索", notes = "根据学生属性进行搜索")
	public AjaxResult list(@RequestBody Student req) {
		AjaxResult ajaxResult = new AjaxResult();
		List<Student> rows = new ArrayList<Student>();
		if (req.getId() == null && req.getName() == null
				&& req.getAge() == null && req.getSex() == null
				&& req.getGrade() == null) {
			rows = list;
		} else {
			for (Student item : list) {
				if (item.getId() == req.getId()) {
					rows.add(item);
					continue;
				}
				if (req.getName() != null
						&& item.getName().indexOf(req.getName()) != -1) {
					rows.add(item);
					continue;
				}
				if (item.getAge() == req.getAge()) {
					rows.add(item);
					continue;
				}
				if (item.getSex().equals(req.getSex())) {
					rows.add(item);
					continue;
				}
				if (req.getGrade() != null
						&& item.getGrade().indexOf(req.getGrade()) != -1) {
					rows.add(item);
					continue;
				}
			}
		}

		ajaxResult.setRows(rows);
		ajaxResult.setTotal((long) rows.size());
		return ajaxResult;
	}

	/**
	 * 根据ID主键获取学生
	 * 
	 * @return
	 * @author lincl
	 * @date 2016年8月26日 下午2:55:56
	 */
	@RequestMapping(value = "/{id}", method = RequestMethod.GET)
	@ApiOperation(value = "根据ID主键获取学生", notes = "根据ID主键获取学生")
	public AjaxResult getStudent(@PathVariable Integer id) {
		AjaxResult ajaxResult = new AjaxResult();
		for (Student item : list) {
			if (id == item.getId()) {
				ajaxResult.setResult(item);
				return ajaxResult;
			}
		}
		ajaxResult.setCode(StatusCode.APP_1004);
		return ajaxResult;
	}

	/**
	 * 根据ID主键删除学生
	 * 
	 * @param id
	 * @return
	 * @author lincl
	 * @date 2016年8月26日 下午3:51:20
	 */
	@RequestMapping(value = "/delete/{id}", method = RequestMethod.DELETE)
	@ApiOperation(value = "根据ID主键删除学生", notes = "根据ID主键删除学生")
	public AjaxResult delete(@PathVariable Integer id) {
		for (Student item : list) {
			if (id == item.getId()) {
				list.remove(item);
				return new AjaxResult();
			}
		}
		return new AjaxResult(StatusCode.APP_1004);
	}

	/**
	 * 新增学生实例
	 * 
	 * @param entity
	 * @return
	 */
	@RequestMapping(value = "/add", method = RequestMethod.POST)
	@ApiOperation(value = "新增学生实例", notes = "新增学生实例")
	public AjaxResult add(@RequestParam String name, @RequestParam String sex,
			@RequestParam Integer age, @RequestParam String grade) {
		Student student = new Student(id++, name, sex, age, grade);
		list.add(student);
		AjaxResult ajaxResult = new AjaxResult();
		ajaxResult.setResult(student.getId());
		return ajaxResult;
	}

	/**
	 * 根据ID更新学生实例
	 * 
	 * @param entity
	 * @param id
	 * @return
	 */
	@RequestMapping(value = "/update/{id}", method = RequestMethod.PUT)
	@ApiOperation(value = "根据ID更新学生实例", notes = "根据ID更新学生实例")
	public AjaxResult update(@RequestParam(required = false) String name,
			@RequestParam(required = false) String sex,
			@RequestParam(required = false) Integer age,
			@RequestParam(required = false) String grade,
			@PathVariable Integer id) {

		Student entity = null;
		for (Student item : list) {
			if (id == item.getId()) {
				entity = item;
			}
		}

		if (entity == null) {
			return new AjaxResult(StatusCode.APP_1004);
		} else {
			if (DataUtil.isNotNull(name)) {
				entity.setName(name);
			}
			if (DataUtil.isNotNull(sex)) {
				entity.setSex(sex);
			}
			if (age != null) {
				entity.setAge(age);
			}
			if (DataUtil.isNotNull(grade)) {
				entity.setGrade(grade);
			}
			return new AjaxResult();
		}
	}

}
```

### 启动tomcat，访问 http://localhost:8080/tiana-api/swagger/index.html

![image](http://note.youdao.com/yws/public/resource/5e6a454ab7a654d9db79e939a65bb8f7/xmlnote/9A8E81E61D90485190B7E2634E2E2804/9087)
