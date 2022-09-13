# JVM—类加载

## 类加载阶段

### 1.加载

- 将类的字节码载入方法区中

- 如果这个类还有父类没有加载则先加载父类

- 加载和链接可能是交替运行的

  ![image-20220913165151636](JVM—类加载.assets/image-20220913165151636.png)

### 2.链接

1. 验证

   - 验证类是否符合JVM规范，安全性检查

2. 准备

   为static变量分配空间，设置默认值

   - static变量分配空间和赋值是两个步骤，分配空间在准备阶段完成，赋值在初始化阶段完成
   - 如果static变量是final的基本类型，以及字符串常量，那么编译阶段值就确定了，赋值在准备阶段完成
   - 如果static变量是final的，但不属于引用类型，那么赋值也会在初始化阶段完成

3. 解析

   - 将常量池中的符号引用解析为直接引用

   ```java
   public class Load2 {
       public static void main(String[] args) throws ClassNotFoundException, IOException {
           // loadClass 方法不会导致类的解析和初始化
           ClassLoader classloader = Load2.class.getClassLoader();
           Class<?> c = classloader.loadClass("cn.itcast.jvm.t3.load.C");
   //        new C();
           System.in.read();
       }
   }
   
   class C {
       D d = new D();
   }
   
   class D {
   
   }
   ```

### 3.初始化

