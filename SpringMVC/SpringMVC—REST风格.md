# SpringMVC—REST风格

## 简介

- 传统风格

  http://localhost/user/getById?id=1

  http://localhost/user/saveUser

- REST风格

  http://localhost/user/1

  http://localhost/user

- 优点

  - 隐藏资源的访问行为，无法通过地址得知对资源是何种操作
  - 书写简化

- 使用行为动作区分对资源进行了何种操作

  | 路径                     | 功能             | 动作               |
  | ------------------------ | ---------------- | ------------------ |
  | http://localhost/users   | 查询全部用户信息 | GET（查询）        |
  | http://localhost/users/1 | 查询指定用户信息 | GET（查询）        |
  | http://localhost/users   | 添加用户信息     | POST（新增、保存） |
  | http://localhost/users   | 修改用户信息     | PUT（修改、更新）  |
  | http://localhost/users/1 | 删除用户信息     | DELETE（删除）     |

- 根据REST风格对资源进行访问称为RESTful

## 使用

```java
//设置当前操作的访问路径
    @RequestMapping(value = "/users",method = RequestMethod.POST)
    //设置当前操作的返回值类型
    @ResponseBody
    public String save(){
        System.out.println("user save...");
        return "{'module':'springmvc'}";
    }

    @RequestMapping(value = "/users/{id}",method = RequestMethod.DELETE)
    @ResponseBody
    public void delete(@PathVariable Integer id){
        System.out.println("user delete..."+id);
    }

    @RequestMapping(value = "/users",method = RequestMethod.PUT)
    @ResponseBody
    public void update(){
        System.out.println("user update...");
    }
```

## 简化开发

```java
//@Controller
//@ResponseBody
@RestController
@RequestMapping("/books")
public class BookController {
    //设置当前操作的访问路径
//    @RequestMapping(method = RequestMethod.POST)
    @PostMapping
    public String save(@RequestBody Book book){//调用时传入json数据
        System.out.println("book save...");
        return "{'module':'springmvc'}";
    }

//    @RequestMapping(value = "/{id}",method = RequestMethod.DELETE)
    @DeleteMapping("/{id}")
    public void delete(@PathVariable Integer id){
        System.out.println("book delete..."+id);
    }

//    @RequestMapping(method = RequestMethod.PUT)
    @PutMapping
    public void update(){
        System.out.println("book update...");
    }
//    @RequestMapping(method = RequestMethod.GET)
    @GetMapping
    public void getAll(){
        System.out.println("book getAll");
    }
}
```

## 设置对静态资源的访问放行

```java
@Configuration
public class SpringMvcSupport extends WebMvcConfigurationSupport {
    @Override
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        //当访问/pages下的文件时，走/pages目录下的内容
        registry.addResourceHandler("/pages/**").addResourceLocations("/pages/");
        registry.addResourceHandler("/js/**").addResourceLocations("/js/");
        registry.addResourceHandler("/css/**").addResourceLocations("/css/");
        registry.addResourceHandler("/plugins/**").addResourceLocations("/plugins/");
    }
}
```

