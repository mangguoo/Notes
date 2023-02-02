## Java程序基础

### Java程序的基本结构

```java
/**
 * 可以用来自动创建文档的注释
 */
public class Hello {
    public static void main(String[] args) {
        // 向屏幕输出文本:
        System.out.println("Hello, world!");
        /* 多行注释开始
        注释内容
        注释结束 */
    }
} // class定义结束
```

1. 注意到`public`是访问修饰符，表示该`class`是公开的。不写`public`，也能正确编译，但是这个类将无法从命令行执行**类名要求：**
   - 类名必须以英文字母开头，后接字母，数字和下划线的组合
   - 习惯以大写字母开头

2. 方法名是`main`，返回值是`void`，表示没有任何返回值，而关键字`static`是另一个修饰符，它表示静态方法，Java入口程序规定的方法必须是静态方法，方法名必须为`main`，括号内的参数必须是String数组**函数命名要求：**
   - 命名和`class`一样，但是首字母小写

3. 在方法内部，语句才是真正的执行代码。Java的每一行语句必须以分号结束

4. 在Java程序中，注释是一种给人阅读的文本，不是程序的一部分，所以编译器会自动忽略注释。Java有3种注释：

   - 单行注释

   ```java
   // 这是注释...
   ```

   - 多行注释

   ```java
   /*
   这是注释
   blablabla...
   这也是注释
   */
   ```

   - 还有一种特殊的多行注释，以`/**`开头，以`*/`结束，如果有多行，每行通常以星号开头，这种特殊的多行注释需要写在类和方法的定义处，可以用于自动创建文档:

   ```java
   /**
    * 可以用来自动创建文档的注释
    * 
    * @auther liaoxuefeng
    */
   public class Hello {
       public static void main(String[] args) {
           System.out.println("Hello, world!");
       }
   }
   ```

