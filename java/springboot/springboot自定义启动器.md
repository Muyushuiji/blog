---
title: springboot starter
date: 2023-02-17 17:09:00
categories:
- java
tags:
- springboot 
---

## Starter的定义

Starter是一种对依赖的合成

在没有Starter之前，Spring使用jpa的操作

1. Maven引入数据库和Jpa的依赖
2. 在xml钟配置属性信息
3. 调试直到运行成功。

每次新建一个Jpa项目都需要重复上述操作。这样配置的时候浪费时间，过程繁琐的话每一步操作都会提高错误率。

### 使用Starter

>starter的主要目的是为了解决上面的效率问题。

#### Starter用途理念

starter把所有用到的依赖都给包含进去，避免开发者自己去引入依赖带来的麻烦。不同的starter内部实现有很大的差异，因为都是为了解决不同的依赖。

#### Starter的实现

starter使用到的相同内容：`CnfigurationProperties`和`AutoConfiguration`。

`CnfigurationProperties`保存配置信息，`AutoConfiguration`使用其中保存的配置信息和配置依赖需要用到的jar。

#### Starter的作用

帮助用户简化了配置的操作。可以给一个组件创建一个starter来让用户在使用这个组件的时候更加简单方便。

#### 如何编写自动配置

> 规则 自定义自动配置类的注解

```
@Configuration //指定这个类是一个配置类
@ConditionalOnXXX //在指定条件成立的情况下自动配置类生效
@AutoConfigureAfter //指定自动配置类的顺序
@Bean //给容器中添加组件
@ConfigurationPropertie//结合相关xxxProperties类来绑定相关的配置
```

>将需要启动就加载的自动配置类，配置在META-INF/spring.factories下

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
```

>模式

`Starter`启动器只用来做依赖注入，写一个自动配置模块，启动器依赖自动配置。别人使用时只需要引入启动器即可，一般自定义启动器名为：`xxx-spring-boot-starter`

>启动器

启动器模块是一个空JAR文件，仅提供辅助性依赖管理，这些依赖可能用于自动装配或者其他类库。

#### 自定义`Starter`

1.  创建一个`starter`项目
2. 创建一个`ConfigurationProperties`保存配置信息（项目不使用配置信息可跳过，大部分都需要）
3. 创建一个`AutoConfiguration`，引用定义好的配置信息；在`AutoConfiguration`钟实现`starter`应该完成的操作，把这个类加入spring.factories配置文件中进行声明
4. 打包项目，在一个Springboot项目中引入该项目依赖，就可以使用该starter了。

##### 创建`autoconfiguration`项目

* 导入POM

```xml
<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.6.3</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.example</groupId>
	<artifactId>httpclient-spring-boot-autoconfiguration</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>


		<dependency>
			<groupId>org.apache.httpcomponents</groupId>
			<artifactId>httpclient</artifactId>
		</dependency>
		<dependency>
			<groupId>org.apache.httpcomponents</groupId>
			<artifactId>httpcore</artifactId>
		</dependency>
		<dependency>
			<groupId>commons-codec</groupId>
			<artifactId>commons-codec</artifactId>
			<version>1.11</version>
		</dependency>
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-lang3</artifactId>
			<version>3.4</version>
		</dependency>

	</dependencies>
	<build>
		<resources>
			<resource>
				<directory>src/main/resources</directory>
				<includes>
					<include>META-INF/*</include>
				</includes>
			</resource>
		</resources>
	</build>
```

* `Properties类`

```java
//自动获取配置文件中前缀为httprequest的属性，把值传入对象参数
@ConfigurationProperties(prefix = "httprequest")
public class HttpRequestProperties {
	//是否启动
    private boolean enabled;

    public boolean isEnabled() {
        return enabled;
    }

    public void setEnabled(boolean enabled) {
        this.enabled = enabled;
    }
}
```

* 工具类

```java
public class HttpRequestClient {
    /**
     * 发送POST请求
     *
     * @param url  请求url
     * @param data 请求数据
     * @return 结果
     */
    @SuppressWarnings("deprecation")
    public static String doPost(String url, String data) {
        CloseableHttpClient httpClient = HttpClients.createDefault();
        HttpPost httpPost = new HttpPost(url);
        RequestConfig requestConfig = RequestConfig.custom()
                .setSocketTimeout(10000).setConnectTimeout(20000)
                .setConnectionRequestTimeout(10000).build();
        httpPost.setConfig(requestConfig);
        String context = StringUtils.EMPTY;
        if (!StringUtils.isEmpty(data)) {
            StringEntity body = new StringEntity(data, "utf-8");
            httpPost.setEntity(body);
        }
        // 设置回调接口接收的消息头
        httpPost.addHeader("Content-Type", "application/json");
        CloseableHttpResponse response = null;
        try {
            response = httpClient.execute(httpPost);
            HttpEntity entity = response.getEntity();
            context = EntityUtils.toString(entity, HTTP.UTF_8);
        } catch (Exception e) {
            e.getStackTrace();
        } finally {
            try {
                response.close();
                httpPost.abort();
                httpClient.close();
            } catch (Exception e) {
                e.getStackTrace();
            }
        }
        return context;
    }

    /**
     * 解析出url参数中的键值对
     *
     * @param url url参数
     * @return 键值对
     */
    public static Map<String, String> getRequestParam(String url) {

        Map<String, String> map = new HashMap<String, String>();
        String[] arrSplit = null;

        // 每个键值为一组
        arrSplit = url.split("[&]");
        for (String strSplit : arrSplit) {
            String[] arrSplitEqual = null;
            arrSplitEqual = strSplit.split("[=]");

            // 解析出键值
            if (arrSplitEqual.length > 1) {
                // 正确解析
                map.put(arrSplitEqual[0], arrSplitEqual[1]);
            } else {
                if (arrSplitEqual[0] != "") {
                    map.put(arrSplitEqual[0], "");
                }
            }
        }
        return map;
    }
}
```

* 自动配置类

```java
@Configuration
@EnableConfigurationProperties(HttpRequestProperties.class)
public class HttpAutoConfiguration {
    
    // 在Spring上下文中创建一个对象
    @Bean
    //只有当httprequest.enabled=true时才创建对象
    @ConditionalOnProperty(prefix = "httprequest", name = "enabled", havingValue = "true")
    @ConditionalOnMissingBean(HttpRequestClient.class)
    public HttpRequestClient getHttpRequestClient() {
        return new HttpRequestClient();
    }
}
```

在resources目录下添加META-INF路径，创建`spring.factories`文件

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.example.httpclientspringbootautoconfiguration.HttpAutoConfiguration
```

配置`HttpAutoConfiguration`自动配置类的信息，配置完成后maven打包（clean、install）

>注意：configuration只保存自定义的配置信息，不需要spring启动类、application配置文件、test测试类。

##### 创建Starter

该项目只导入configuration项目依赖，提供xxx-springboot-starter依赖给用户使用。

* pom.xml

```xml
<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.6.3</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>httpclient-springboot-starter</groupId>
	<artifactId>httpclient-springboot-starter</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>com.example</groupId>
			<artifactId>httpclient-spring-boot-autoconfiguration</artifactId>
			<version>0.0.1-SNAPSHOT</version>
		</dependency>
	</dependencies>
```

#### 创建项目测试Starter

* pom.xml

```xml
<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.6.3</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.example</groupId>
	<artifactId>http-auto-config-test</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>http-auto-config-test</name>
	<description>Demo project for Spring Boot</description>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

        <!--引入自定义的starter-->
		<dependency>
			<groupId>httpclient-springboot-starter</groupId>
			<artifactId>httpclient-springboot-starter</artifactId>
			<version>0.0.1-SNAPSHOT</version>
		</dependency>
	</dependencies>
```

* 配置信息

```properties
httprequest.enabled=true
```

* 启动类和测试类

```java
@SpringBootApplication
public class HttpAutoConfigTestApplication {

	public static void main(String[] args) {
		SpringApplication.run(HttpAutoConfigTestApplication.class, args);
	}

}
```

```java
@SpringBootTest
class HttpAutoConfigTestApplicationTests {

	@Resource
	private HttpRequestClient httpRequestClient;
	@Test
	public void test() {
		String str = httpRequestClient.doPost("http://www.baidu.com", null);
		System.out.println(str);
	}
}
```

只有当设置httprequest.enabled=true时启动项目会创建httpRequestClient对象

当设置httprequest.enabled=false时
