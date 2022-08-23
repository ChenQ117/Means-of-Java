# MyBatis

## 核心配置文件

![image-20220823180329157](MyBatis.assets/image-20220823180329157.png)

<font color='red'>配置各个标签时需要遵守前后顺序。</font>

### environments(配置数据库环境)

- 配置数据库连接环境信息，可以配置多个environment，通过default属性切换不同的environment

```xml
	<environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql:///myfriend?useSSL=false"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
```

## mappers(映射代理)

- 配置sql映射映射代理

  ```xml
      <mappers>
  <!--        加载sql映射文件-->
  <!--        <mapper resource="com/cq/mapper/UserMapper.xml"/>-->
  <!--        Mapper代理方式-->
          <package name="com.cq.mapper"/>
      </mappers>
  ```

## typeAliasee(类别名)

- 配置别名

  ```xml
  <!--    给pojo下的文件配置别名，在映射中就可以不写包名了，而且类名还不区分大小写-->
      <typeAliases>
          <package name="com.cq.pojo"/>
      </typeAliases>
  ```

  