[toc]

# 1 变量

## 变量声明

```java
int i = 10;
```



## 变量类型

**基本数据类型**：

- 整数类型：`byte, short, int, long`
- 浮点类型：`float, double`

- 字符型：`char`
- 布尔型：`boolean`

**引用数据类型**：

- 类：`class`
- 接口：`interface`
- 数组：`[]`
- 字符串：`String`



## 运算符

**算数运算符：**

- `+, -, *, /, %`

- `+`（`String` 中的连接符）

- `++, --`

**赋值运算符：**

- `=`

- `+=, -=, *=, /=, %=`

**比较运算符：**

- `==, !=, >=, <=, >, <`

- `instanceof`：测试它左边的对象是否是它右边的类的实例，返回 boolean 的数据类型

**逻辑运算符：**

- `&, |, !, &&, ||, ^(异或)`

**三元运算符：**

```java
(1 > 2) ? "1>2" : "1<2"
```



## 字符串

- **连接运算符 `+`**，对字符串进行拼接；**`+` 两侧可以是任意类型，最终结果一定是 String 类型**





# 2 向控制台输出

```java
System.out.println("Hello World");
```





# 3 集合

## 数组

- **数组的动态初始化：**

  数组的空间分配与初始化分开进行。比如以下代码，

  ```java
  int[] arr2 = new int[3];
  arr2[0] = 1;
  arr2[1] = 2;
  ```

- **数组的静态初始化：**

  数组的空间分配与初始化同步进行。比如以下代码，

  ```java
  //方法1：
  int[] arr1 = {1, 2, 3};
  //方法2：
  int[] arr3 = new int[]{1, 2, 3};
  ```

- **数组元素的调用：**

  使用下标调用。比如以下代码，

  ```java
  arr2[2]
  ```

- **获取数组的长度：**

  ```java
  arr3.length
  ```

- **一维数组的遍历：**

  ```java
  int[] arr = {1, 2, 3};
  for (int i = 0; i < arr.length; i++) {
      System.out.println(arr[i]);
  }
  ```





# 4 流程控制

## `if-else` 语句

```java
Scanner scanner = new Scanner(System.in);
int i = scanner.nextInt();
if (i > 20) {
    System.out.println("i>20");
} else if (i > 10) {
    System.out.println("20>=i>10");
} else {
    System.out.println("i<=10");
}
```



## `for` 语句

```java
int sum = 0;
for (int i = 1; i < 11; i++) {
    sum += i;
}
System.out.println(sum);

// for-each循环
List<String> strings = Arrays.asList("apple", "banana", "cherry");
for (String fruit : strings) {
    System.out.println(fruit);
}
```



## `while` 语句

```java
int sum = 0;
int i = 1;
while (i <= 10) {
    sum += i;
    i++;
}
System.out.println(sum);
```





# 5 对象

## 声明, 创建对象

- **定义类：**

  ```java
  class test3 {
      public int i = 10; //属性
  
      public int test3_method() { //方法
          return i;
      }
  }
  ```

- **创建对象：**

  ```java
  test3 test3 = new test3(); //创建对象
  System.out.println(test3.test3_method()); //调用方法
  ```

- **匿名对象：**

  ```java
  new test3().test3_method()
  ```

- **可变参数：**

  将<u>*可变参数看做数组操作*</u>

  可变参数<u>*只能放在参数列表的最后*</u>

  ```java
  class test3 {
      public void multi_args(int... args) {
          for (int arg : args) {
              System.out.println(arg);
          }
      }
  }
  ```

  传入多个参数。比如以下代码，

  ```java
  new test3().multi_args(3, 4, 5);
  ```

- **参数传递：**

  java 的参数传递机制是，**值传递**。即将值复制一份，作为实参传入

  



## 构造器

- **构造器：**

  <u>*构造器与类具有相同的名称*</u>

  构造器不声明返回值类型；即<u>*没有 `return` 语句*</u>

  ```java
  class A {
      public int i;
      
      // 无参构造器
      public A() {
  
      }
  
      // 有参构造器
      public A(int i) {
          this.i = i;
      }
  }
  ```

  <u>*默认提供一个无参构造器*</u>；如果显示定义了构造器，那么不再提供默认构造器

- **`this` 关键字：**

  ```java
  class A {
      public int i = 10;
  
      public A(int i) {
          this.i = i;
      }
  }
  ```



## 继承

通过 **`extends` 关键字**实现继承；**单继承**

```java
class Human {
    public int age;
    private String sex;
}

class Student extends Human {
    public String stu_id;
}
```

**方法的重写：**

```java
class Human {
    public int age;
    public String sex;

    void method(String s) {
        System.out.println(s);
    }
}

class Student extends Human {
    public String stu_id;

    @Override
    // 重写方法
    public void method(String s) {
        System.out.println("student:" + s);
    }
}
```

**`super` 关键字：**

<u>*子类中所有的构造器，都会访问父类的空参构造器*</u>；相当于子类每个构造器的第一行，都有个隐藏的 `super()`；如果父类中没有空参构造器，那么就要显式使用 <u>*`super(arg)` 调用父类的有参构造器*</u>

```java
class Creature{
    public Creature(int i){ //父类不提供空参构造器
    }
}

class Human extends Creature{
    public Human(){ //父类没有空参构造器，所以要显式调用父类某个存在的构造器
        super(10);
    }
    
    public Human(int i){ //调用自己的空参构造器，而其中显式调用了父类的构造器，是合法的
        this();
    }
}
```



## 多态

**编译看左边，执行看右边**

子类到父类的类型转换（即将子类赋值给父类）自动进行；父类到子类的转换（即将父类赋值给子类）通过强制类型转换

```java
public class test2 {
    public static void main(String[] args) {
        new Utils().test(new B()); //形参是A类，但是也可以传入A类的子类，B类
    }
}

class Utils {
    public void test(A a) {
        a.method(); //执行看右边，执行B类的method()方法，输出，B
//        a.methodB(); //编译看左边，A类中没有methodB()方法，IDE报错，编译不通过
        if (a instanceof B) {
            ((B) a).methodB();
        }
    }
}

class A {
    public void method() {
        System.out.println("A");
    }
}

class B extends A {
    @Override
    public void method() {
        System.out.println("B");
    }

    public void methodB(){
        System.out.println("methodB");
    }
}

class C extends A {
    @Override
    public void method() {
        System.out.println("C");
    }
}
```



## `Object` 类

`Object` 是所有类的父类，有一些方法可以<u>*进行重载*</u>

- **`equals()` 方法：**

  `Object` 类中的 `equals()` 方法如下，

  ```java
  public boolean equals(Object obj) {
  	return (this == obj);
  }
  ```

  **默认 `equals()` 方法比较的是地址值**

- **`toString()` 方法：**

  默认打印的是地址值



## abstract 和 interface

**关键字 `abstract`**

- 含有抽象方法的类必须声明为抽象类
- **抽象类无法直接被实例化**，但是可以通过多态实例化
- 继承抽象类的类，如果没有重写所有的抽象方法，则仍然是个抽象类

```java
public class test1 {
    public static void main(String[] args) {
        A a = new B(); //可以通过多态实例化
    }
}

abstract class A { //抽象类
    public abstract void abstractMethod(); //抽象方法
}

class B extends A{
    @Override
    public void abstractMethod() {
        
    }
}
```

---

**关键字 `interface`** 

- 实现接口的类，除非用 `abstract` 修饰，否则必须重写接口中的抽象方法（和抽象类的继承规则是一致的）
- 接口无法直接实例化，但是可以通过多态实例化
- **类可以实现多个接口**
- **接口不可以实现任何接口**
- **接口可以继承多个接口**；但<u>*不能*</u>继承类

```java
interface C {
    int i = 10; //常量，相当于public final static int i = 10;

    void TypeC(); //抽象方法
}

interface A {
    void USB();

}

class B implements A, C { // 类B实现了接口A和接口C
    @Override
    public void USB() {
    }

    @Override
    public void TypeC() {
    }
}
```





# 7 文件, 流

流可以分为**节点流**和**处理流**：**节点流的处理对象是文件**，而**处理流的处理对象是流**

**常用的节点流：**`FileInputStream、FileOutputStream、FileReader、FileWriter`

**常用的处理流：**`BufferedInputStream、BufferedOutputStream、BufferedReader、BufferedWriter、InputStreamReader、OutPutStreamReader、ObjectInputStream、ObjectOutputStream`



## 节点流

`FileReader、FileWriter` 读写单位是字符，`FileInputStream、FileOutputStream` 读写单位是字节

**`FileReader` 类：**

1. 处理对象是 `File` 对象
2. `fileReader.read(chars)` 方法表示一次性读取指定数组长度的数据，返回读取字符的长度
3. `new String(chars, offset, len)` 构造器通过读取指定长度，保证了数组中的脏数据不会被读入

以下代码使用 `FileReader` 类实现了文件的读入，

```java
@Test
public void testFileReader() throws IOException {
    File file = new File("src/module1/test1.txt");
    FileReader fileReader = new FileReader(file);
    int readLen;
    char[] chars = new char[1024];
    while (((readLen = fileReader.read(chars)) != -1)) {
        System.out.println(new String(chars, 0, readLen));
    }
    fileReader.close();
}
```

**`FileWriter` 类：**

1. `new FileWriter(file, append)` 构造器，第一个参数是 `File` 对象；第二个参数，`true` 表示在已有文件后追加，`false` 表示直接覆盖

以下代码使用 `FileWriter` 类实现了文件的写入，

```java
@Test
public void testFileWriter() throws IOException {
    File file = new File("src/module1/test2.txt");
    FileWriter fileWriter = new FileWriter(file, false);
    fileWriter.write("hello world\n");
    fileWriter.write("hello python");
    fileWriter.close();
}
```

**`FileInputStream` 和 `FileOutputStream`：**

以下代码使用 `FileInputStream、FileOutputStream` 类实现了文件的复制，

```java
@Test
public void testFileInOutputStream() throws IOException {
    File current = new File("src/module1/pic1.png");
    File copy = new File("src/module1/pic2.png");
    FileInputStream fileInputStream = new FileInputStream(current);
    FileOutputStream fileOutputStream = new FileOutputStream(copy);
    int readLen;
    byte[] bytes = new byte[1024];
    while ((readLen = fileInputStream.read(bytes)) != -1) {
        fileOutputStream.write(bytes, 0, readLen);
    }
    fileInputStream.close();
    fileOutputStream.close();
}
```



## 处理流

**`BufferedInputStream` 和 `BufferedOutputStream`：**

1. 缓冲流可以提高效率
2. 缓冲流的处理对象是流
3. **关闭外层的缓冲流就可以关闭里层的流**

```java
BufferedInputStream bufferedInputStream = new BufferedInputStream(new FileInputStream(current));
BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(new FileOutputStream(copy));
```

4. 缓冲流的实现在于其内部的缓冲区，缓冲区满后才进行一次操作；`buffer.flush()` 方法可以强制执行一次操作

**`BufferedReader` 和 `BufferedWriter`：**

1. `reader.readLine()` 方法，一次读取一行，不读取换行符
2. `writer.newLine()` 方法，相当于添加换行符

以下代码使用缓冲流实现了文件的复制，

```java
@Test
public void testBufferedWriterReader() throws IOException {
    File current = new File("src/module1/test1.txt");
    File copy = new File("src/module1/test2.txt");
    BufferedReader bufferedReader = new BufferedReader(new FileReader(current));
    BufferedWriter bufferedWriter = new BufferedWriter(new FileWriter(copy));
    String read_str;
    while ((read_str = bufferedReader.readLine()) != null) {
        bufferedWriter.write(read_str);
        bufferedWriter.newLine();
    }
    bufferedReader.close();
    bufferedWriter.close();
}
```





# 8 包

```java
import java.util.*;
```





# 9 异常处理

## `try-catch-fianlly` 结构

```java
public class test2 {
    public static void main(String[] args) {
        test2 t = new test2();
        System.out.println(t.exceptionTest()); //输出，3
    }
    
    public int exceptionTest(){
        int[] a = new int[]{1, 2, 3};
        try {
            System.out.println(a[3]); //数组越界异常
            System.out.println("before exception"); //因为上一行代码出现了异常，所以这一句不会执行
            return 0; //return语句出现在了try代码块中
        } catch (IndexOutOfBoundsException e) { //可以捕获多个异常，但最详细的异常必须最先被捕获
            return 1; //catch代码块中也要有return语句
        } catch (Exception e) { //不会执行，因为IndexOutOfBoundsException是Exception的子类, 已经在上一个catch块中被捕获了
            return 2;
        } finally {
            return 3;
        }
    }
}
```



## 关键字 `throws`

- 不直接报错，而是**交给外层处理**；如果外层也处理不了再报错

```java
public class ThrowsExample {

    // 这个方法声明它可能会抛出IOException
    public static void readFile() throws IOException {
        FileReader file = new FileReader("example.txt");
        int i;
        while ((i = file.read()) != -1) {
            System.out.print((char) i);
        }
        file.close();
    }

    public static void main(String[] args) {
        try {
            // 在这里调用可能会抛出异常的方法
            readFile();
        } catch (IOException e) {
            // 处理IOException
            System.out.println("An error occurred while reading the file: " + e.getMessage());
        }
    }
}

```

