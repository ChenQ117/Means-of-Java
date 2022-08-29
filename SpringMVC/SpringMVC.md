# SpringMVC

- SpringMVC技术与Servlet技术功能相同，均属于web层开发技术

- SpringMVC是一种基于Java实现MVC模型的轻量级Web框架
- 优点：
  - 使用简单，开发便捷（相比于Servlet）
  - 灵活性强

## 简单使用

1. 使用SpringMVC技术需要先导入SpringMVC坐标与Servlet坐标

   ```xml
     <dependencies>
   <!--    1.导入坐标springmvc与Servlet-->
       <dependency>
         <groupId>javax.servlet</groupId>
         <artifactId>javax.servlet-api</artifactId>
         <version>3.1.0</version>
         <scope>provided</scope>
       </dependency>
       <dependency>
         <groupId>org.springframework</groupId>
         <artifactId>spring-webmvc</artifactId>
         <version>5.2.10.RELEASE</version>
       </dependency>
     </dependencies>
   
   <!--  导入tomcat坐标-->
     <build>
       <plugins>
         <plugin>
           <groupId>org.apache.tomcat.maven</groupId>
           <artifactId>tomcat7-maven-plugin</artifactId>
           <version>2.1</version>
           <configuration>
             <port>80</port>
             <path>/</path>
           </configuration>
         </plugin>
       </plugins>
     </build>
   ```

2. 创建SpringMVC控制器类（等同于Servlet功能）

   ```java
   @Controller
   public class UserController {
       //设置当前操作的访问路径
       @RequestMapping("/save")
       //设置当前操作的返回值类型
       @ResponseBody
       public String save(){
           System.out.println("user save...");
           return "{'module':'springmvc'}";
       }
   }
   ```

3. 初始化SpringMVC环境（同Spring环境），设定SpringMVC加载对应的bean

   ```java
   @Configuration
   @ComponentScan("com.cq.controller")
   public class SpringMvcConfig {
   
   }
   ```

4. 初始化Servlet容器，加载SpringMVC环境，并设置SpringMVC技术处理的请求

   ```java
   public class ServletContainerInitConfig extends AbstractDispatcherServletInitializer {
   
       //加载SpringMVC容器配置
       @Override
       protected WebApplicationContext createServletApplicationContext() {
           AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
           ctx.register(SpringMvcConfig.class);
           return ctx;
       }
   
       //设置哪些请求归属SpringMVC处理
       @Override
       protected String[] getServletMappings() {
           return new String[]{"/"};
       }
   
       //加载spring容器配置
       @Override
       protected WebApplicationContext createRootApplicationContext() {
           return null;
       }
   }
   ```

## AbstractDispatcherServletInitializer

- `AbstractDispatcherServletInitializer`类是SpringMVC提供的快速初始化Web3.0容器的抽象类
- 提供三个接口方法供用户实现
  - `createServletApplicationContext()`，创建Servlet容器时，加载SpringMVC对应的bean并放入`WebApplicationContext`对象范围中，而`WebApplicationContext`的作用范围为`ServletContext`范围，即整个web容器范围
  - `getServletMappings()`，设置SpringMVC对应的请求映射路径，设置为`/`表示拦截所有请求，任意请求都将转入到SpringMVC进行处理
  - `createRootApplicationContext()`，如果创建Servlet容器时需要加载非SpringMVC对应的bean，使用当前方法进行，使用方式同`createServletApplicationContext()`