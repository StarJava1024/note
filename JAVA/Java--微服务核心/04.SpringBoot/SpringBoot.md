# Spring Boot

## 原理初探

### 自动配置

#### 1. pom.xml

- spring-boot-dependencies: 核心依赖在父工程中！
- 我们在写或者引入一些Springboot依赖的时候，不需要指定版本，就因为有这些版本仓库

#### 2. 启动器

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
```

- 启动器：就是Springboot的启动场景；
- 比如spring-boot-starter-web，他就会帮我们自动导入web环境所有的依赖！
- springboot会将所有的功能场景，都变成一个个的启动器
- 我们要使用什么功能，就只需要找到对应的启动器就可以了`starter`

#### 3.主程序

```Java
// @SpringBootApplication: 标注这个类是一个 springboot 的应用
@SpringBootApplication
public class SpringbootTestApplication {

    public static void main(String[] args) {
        // 将springboot应用启动
        SpringApplication.run(SpringbootTestApplication.class, args);
    }
    
}
```

- 注解

    - ```
        @SpringBootConfiguration: springboot的配置
        	@Configuration: spring配置类
        		@Component: spring组件
        @EnableAutoConfiguration: 自动配置
        	@AutoConfigurationPackage: 自动配置包
        		@Import(AutoConfigurationPackages.Registrar.class): 自动配置`包注册`
        	@Import(AutoConfigurationImportSelector.class): 自动配置导入选择
        ```

#### 4. yaml

```yaml
# 普通的 key-value
name: wx
# 对象
student:
	name: wx
	age: 20
# 行内写法
student: {name: wx,age: 20}
# 数组
student:
	- name
	- age
student: [name,age]
```

### SpringBoot Web开发

#### 1. 静态资源

- 在SpringBoot，我们可以使用以下方式处理静态资源
    - webjars `locahost:8080/webjars`
    - resource，static，public，/** `locahost:8080/`
- 优先级：resource > static(默认) > public

#### 2. 定制首页

- 导入静态资源……

- 首页

- jsp，模板引擎 Thymeleaf

    >只需要使用 thymeleaf，只需要导入对应的依赖就可以了！我们将HTML放在外面的templates目录下即可！
    >
    >```Java
    >public static final String DEFAULT_PREFIX = "classpath:/templates";
    >public static final String DEFAULT_PREFIX = ".html";
    >```

- 装配扩展 SpringMVC

    >在springboot中，有非常多的xxxConfiguration 帮助我们进行扩展配置，只要看见了这个东西，我们就要注意了！

- 增删改查

- 拦截器

- 国际化