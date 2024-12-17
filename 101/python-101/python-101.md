[toc]

# 1 变量

## 变量声明

```python
a = 101
```

- 变量**必须**赋初值



## 运算符

- **加 `+`，减 `-`，乘 `*`，除 `/`，取余 `%`，乘方 `**`，求商 `//`**

- **三目运算符**

  ```python
  max = a if a>b else b
  ```

- **相等 `==`，大于等于 `>=`，小于等于 `<=`，大于 `>`，小于 `<`，不等于 `!=`**



## 字符串

```python
message = 'hello world'
```

- 可以**单引号**，也可以**双引号**
- Python 中，可以用 **`+` 拼接字符串和字符串**，但不可以拼接字符串和其它类型





# 2 向控制台输出

```python
print('hello world')
```





# 3 集合

## 列表 `list`

**列表的声明：**

```python
l = [101, 201, 301]
```

**列表的增删查改：**

```python
i, j = l[0], l[-1] # 查, 改
l.append(401) # 增
deleted_ele = l.pop(1) # 删
```

**列表的长度：**

```python
length_l = len(l)
```

**列表的切片：**

```python
sliced_l = l[0:2]
```

**列表的合并：**

```python
['hello'] + ['world', '!'] # ['hello', 'world', '!']
```

**列表推导式：**

```python
x = (1, 2, 3)
arr = [i**2 for i in x if i%2 != 0] # [1, 9]
```

**判断元素是否在列表中：**

```python
101 in l # True
```

**列表的排序：**

```python
new_l = sorted(l) # 不影响原列表
l.sort() # 影响原列表
```



## 元组 `tuple`

**元素的声明：**

```python
t = (101, 201, 301)
```

- **元组是不可变的列表**



## 字典 `dict`

**字典的声明：**

```python
d = {"key1": "value1", "key2": "value2"}
```

**字典的增删查改：**

```python
# 查, 改
v = d["key1"]
v = d.get("key1", 101) # 第二个参数是默认值
d.items() # [("key1", "value1"), ("key2", "value2")]
d.keys()
d.values()

# 增
d["key3"] = "value3"
```

**字典推导式：**

```python
d = {word: num for word, num in zip(['a', 'b', 'c'], [1, 2, 3])} # {'a': 1, 'b': 2, 'c': 3}
```

**判断键是否在字典中：**

```python
"key1" in d
```



## 拆包

**列表的拆包：**

```python
def sum(a, b):
    print(a + b)
    
l = [1, 2]
sum(*l) # 相当于sum(1, 2)
```

**字典的拆包：**

```python
def f(one, two):
    print(one, two)
    
dic = {'one':1, 'two':2}
f(**dic) # 相当于f(one=1, two=2)
```





# 4 流程控制

## `if-elif-else` 结构

```python
score = 75
if score >= 90:
    print('excellent')
elif score >= 60 and score < 90:
    print('pass')
else:
    print('fail')
```



## `for` 循环

```python
fruits = ['apple', 'banana', 'cherry', 'date']
for fruit in fruits:
    print(fruit)
```



## `while` 循环

```python
list = [1, 2, 3, 1]
while 1 in list:
    list.remove(1)
print(list) # [2, 3]
```





# 5 函数

## 函数的声明

```python
def func1():
	print("hello world!")
```



## 函数的参数

```python
def func1(arg1, arg2, arg3=101):
	print("hello world!")
    
func1(301, arg2=201)
```

**可变参数：**

- 通过 `*` 表示存储在**列表**中

  ```python
  def greet(name, *greetings):
      for greeting in greetings:
          print(name + ' greet ' + greeting)
  
  greet('tom', 'jerry', 'alice') 
  ```

- 通过 `**` 表示存储在**字典**中

  ```python
  def create_user(username, **infos):
      user = {'username': username}
      for key, value in infos.items():
          user[key] = value
      return user
  
  create_user('tom', oldPassword='123', newPassword='666') # 必须通过参数名指定
  ```



## 函数的返回值

```python
def func1(arg1, arg2):
	return arg1, arg2 # 不用指明返回值类型; 可以有多个返回值
    
arg1, arg2 = func1(101, 201) # 相当于先返回了一个tuple, 然后直接拆包
```





# 6 类

## 声明类

```python
class User:
    def __init__(self, username, password): # 构造方法
        self.username = username # 不需要定义属性, 直接绑定到self上就好
        self.password = password
        self.new_comer = 1 # 设置属性的默认值

    def get_username(self): # 如果要使用类中的属性或方法, 必须第一个参数传入self
        return self.username
```



## 实例化类

```python
user = User('tom', '123')
```



## 继承

```Python
class User:    
    def __init__(self, username, password):
        self.username = username
        self.password = password
        self.new_comer = 1
        
    def get_authority(self):
        return None


class Admin(User, Object): # 可以多继承
    def __init__(self, username, password, isAdmin):
        super(Admin, self).__init__(username, password) # 通过super, 调用父类的方法
        self.isAdmin = isAdmin

    def get_authority(self): # 重写方法
        return self.isAdmin
```





# 7 文件

## 读文件

```python
with open('test.txt', 'r') as file: # r表示只读
    for line in file:
        print(line, end="")
```

- `with ... as ...` 语法可以**自动回收资源**



## 写文件

```python
with open('test.txt', 'a') as file: # w表示覆盖, a表示追加
    file.write('I love python\n')
```





# 8 模块

## 导入模块

```python
import math # 导入完整的模块
math.sqrt(101)

from math import sqrt # 导入部分模块
sqrt(101)
```

