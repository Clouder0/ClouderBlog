---
title: "Python 极速入门教程"
date: 2021-05-03T22:44:53+08:00
slug: "python-tutorial"
type: post
tags:
  - python
  - tutorial
categories:
  - Programming
---

## 前言

面向有编程经验者的极速入门指南。

大部分内容简化于 [W3School](https://www.w3schools.com/python/default.asp),翻译不一定准确，因此标注了英文。

包括代码一共两万字符左右，预计阅读时间一小时。

目前我的博客长文显示效果不佳，缺乏目录，因此可以考虑[下载阅读](https://github.com/Clouder0/ClouderBlog/tree/main/content/zh-cn/post/python-tutorial.md)。博客完全开源于 [Github](https://github.com/Clouder0/ClouderBlog).

[toc]

## 语法(Syntax)

文件执行方式：`python myfile.py`

强制缩进，**缩进不能省略**。缩进可以使用任意数量的空格。

```python
if 5 > 2:
 print("Five is greater than two!") 
if 5 > 2:
        print("Five is greater than two!") 
```

### 注释(Comment)

注释语法：

```python
# Single Line Comment
"""
Multiple
Line
Comment
"""
```

## 变量(Variables)

当变量被赋值时，其被创建。
没有显式声明变量的语法。

```python
x = 5
y = "Hello, World!"
```

可以转换类型。

```python
x = str(3)    # x will be '3'
y = int(3)    # y will be 3
z = float(3)  # z will be 3.0
```

可以获得类型。

```python
x = 5
y = "John"
print(type(x))
print(type(y))
```

还可以这样赋值：

```python
x, y, z = "Orange", "Banana", "Cherry"
x = y = z = "Orange"
fruits = ["apple", "banana", "cherry"]
x, y, z = fruits
```

没有在函数中声明的变量一律视作全局变量。

```python
x = "awesome"
def myfunc():
  print("Python is " + x)
myfunc()
```

局部变量优先。

```python
x = "awesome"
def myfunc():
  x = "fantastic"
  print("Python is " + x)
myfunc()
print("Python is " + x)
```

也可以显式声明全局变量。

```python
def myfunc():
  global x
  x = "fantastic"
myfunc()
print("Python is " + x)
```

由于不能区分赋值和声明，因此如果在函数中修改全局变量，需要指明全局。

```python
x = "awesome"
def myfunc():
  global x
  x = "fantastic"
myfunc()
print("Python is " + x)
```

### 数值(Number)

三种数值类型：`int` `float` `complex`

其中复数的虚部用 `j` 来表示。

```python
x = 3+5j
y = 5j
z = -5j

print(type(x))
print(type(y))
print(type(z))
```

### 真值(Boolean)

使用 `True` 和 `False`，大小写敏感。

可以强制转换：

```python
x = "Hello"
y = 15

print(bool(x))
print(bool(y))
```

空值一般转换为假，例如零、空文本、空集合等。

## 条件与循环(If...Else/While/For)

大于小于等于不等于跟 C 语言一致。

如果：

```python
a = 200
b = 33
if b > a:
  print("b is greater than a")
elif a == b:
  print("a and b are equal")
else:
  print("a is greater than b")
```

适当压行也是可以的：

```python
if a > b: print("a is greater than b")
```

三目运算符，需要注意的是执行语句在前面。

```python
a = 2
b = 330
print("A") if a > b else print("B")
print("A") if a > b else print("=") if a == b else print("B")
```

与或：

```python
a = 200
b = 33
c = 500
if a > b and c > a:
  print("Both conditions are True")
if a > b or a > c:
  print("At least one of the conditions is True")
```

如果不能为空，可以传递个 `pass` 占位。

```python
if b > a:
  pass
```

`while` 循环很常规：

```python
i = 1
while i < 6:
  print(i)
  if i == 3:
    break
  if i == 4:
    continue
  i += 1
```

还有个 `else` 的语法：

```python
i = 1
while i < 6:
  print(i)
  i += 1
else:
  print("i is no longer less than 6")
```

这个有什么用呢？其实是可以判断自然结束还是被打断。

```python
i = 1
while i < 6:
  break
else:
  print("i is no longer less than 6")
```

Python 中的 `for` 循环，更像是其他语言中的 `foreach`.

```python
fruits = ["apple", "banana", "cherry"]
for x in fruits:
  if x == "apple":
    continue
  print(x)
  if x == "banana":
    break
```

为了循环，可以用 `range` 生成一个数组。依然是左闭右开。可以缺省左边界 0.

```python
for x in range(6): #generate an array containing 0,1,2,3,4,5
  print(x)
for x in range(2, 6): #[2,6)
  print(x)
```

可以指定步长，默认为 1.

```python
for x in range(2, 30, 3):
  print(x)
```

也支持 `else`

```python
for x in range(6):
  print(x)
else:
  print("Finally finished!")
```

也可以拿 `pass` 占位。

```python
for x in [0, 1, 2]:
  pass
```





## 字符串(String)

有两种写法：

```python
print("Hello")
print('Hello')
```

好像没什么区别。

多行字符串：

```python
a = """Lorem ipsum dolor sit amet,
consectetur adipiscing elit,
sed do eiusmod tempor incididunt
ut labore et dolore magna aliqua."""
print(a)
```

字符串可以直接当数组用。

```python
a = "Hello, World!"
print(a[0])
```

获得长度。

```python
a = "Hello, World!"
print(len(a))
```

直接搜索。

```python
txt = "The best things in life are free!"
print("free" in txt)
print("expensive" not in txt)
if "free" in txt:
  print("Yes, 'free' is present.")
if "expensive" not in txt:
  print("Yes, 'expensive' is NOT present.")
```

几个常用函数：

- `upper`，大写。
- `lower`，小写。
- `strip`，去除两端空格。
- `replace`，替换。
- `split`，以特定分隔符分割。

连接两个字符串，直接用加号。

```python
a = "Hello"
b = "World"
c = a + b
print(c)
```

格式化：

```python
quantity = 3
itemno = 567
price = 49.95
myorder = "I want {} pieces of item {} for {} dollars."
print(myorder.format(quantity, itemno, price))
```

可以指定参数填入的顺序：

```python
myorder = "I want to pay {2} dollars for {0} pieces of item {1}."
print(myorder.format(quantity, itemno, price))
```

转义符：`\`

```python
txt = "We are the so-called \"Vikings\" from the north."
```

## 操作符(Operators)

- 算术运算符
  - `+`
  - `-`
  - `*`
  - `/`
  - `%`，取模。
  - `**`，次幂，例如 `2**10` 返回 $1024$.
  - `//`，向下取整，严格向下取整，例如 `-11//10` 将会得到 $-2$.
- 比较运算符
  - `==`
  - `!=`
  - `>`
  - `<`
  - `>=`
  - `<=`
- 逻辑运算符，使用英文单词而非符号。
  - `and`
  - `or`
  - `not`
- 身份运算符？(Identity Operators)
  - `is`
  - `is not`
  - 用于判断是否为同一个对象，即在内存中处于相同的位置。
- 成员运算符？(Membership Operators)
  - `in`
  - `not in`
  - 用在集合中。
- 位运算符
  - `&`
  - `|`
  - `^`
  - `~`
  - `<<`
  - `>>`

## 集合(Collections)

### 数组(List)

没有 `Array`，只有 `List`.

```python
thislist = ["apple", "banana", "cherry"]
print(thislist)
```

下标从零开始。

```python
thislist = ["apple", "banana", "cherry"]
print(thislist[0])
```

还可以是负的，`-1` 表示倒数第一个，依此类推。

```python
thislist = ["apple", "banana", "cherry"]
print(thislist[-1])
```

获取子数组，左闭右开。例如 `[2:5]` 代表 `[2,5)`

```python
thislist = ["apple", "banana", "cherry", "orange", "kiwi", "melon", "mango"]
print(thislist[2:5])
```

还可以去头去尾。

```python
thislist = ["apple", "banana", "cherry", "orange", "kiwi", "melon", "mango"]
print(thislist[:4])
print(thislist[2:])
```

获得元素个数：

```python
thislist = ["apple", "banana", "cherry"]
print(len(thislist))
```

元素类型都可以不同：

```python
list1 = ["abc", 34, True, 40, "male"]
```

构造：

```python
thislist = list(("apple", "banana", "cherry")) # note the double round-brackets
print(thislist)
```

赋值：

```python
thislist = ["apple", "banana", "cherry"]
thislist[1] = "blackcurrant"
print(thislist)
```

甚至一次改变一个子数组：

```python
thislist = ["apple", "banana", "cherry", "orange", "kiwi", "mango"]
thislist[1:3] = ["blackcurrant", "watermelon"]
print(thislist)
```

元素数量可以不对等，可以视作将原数组中的 `[l,r)`  扔掉，然后从切口塞进去新的子数组。

```python
thislist = ["apple", "banana", "cherry"]
thislist[1:2] = ["blackcurrant", "watermelon"]
print(thislist)
```

支持插入，应该是 $O(n)$ 复杂度的。`insert(x,"something")` 即让 `something` 成为下标为 x 的元素，也就是插入到当前下标为 x 的元素前。

```python
thislist = ["apple", "banana", "cherry"]
thislist.insert(2, "watermelon")
print(thislist)
```

尾部追加，应该是 $O(1)$ 的。

```python
thislist = ["apple", "banana", "cherry"]
thislist.append("orange")
print(thislist)
```

直接连接两个数组：

```python
thislist = ["apple", "banana", "cherry"]
tropical = ["mango", "pineapple", "papaya"]
thislist.extend(tropical)
print(thislist)
```

啥都能连接？

```python
thislist = ["apple", "banana", "cherry"]
thistuple = ("kiwi", "orange")
thislist.extend(thistuple)
print(thislist)
```

删除，一次只删一个。

```python
thislist = ["apple", "banana", "cherry"]
thislist.remove("banana")
print(thislist)
```

按下标删除。可以省略参数，默认删除最后一个。

```python
thislist = ["apple", "banana", "cherry"]
thislist.pop(1)
thislist.pop()
print(thislist)
```

还可以用 `del` 关键字。

```python
thislist = ["apple", "banana", "cherry"]
del thislist[0]
print(thislist)
del thislist #delete the entire list
```

清空，数组对象依然保留。

```python
thislist = ["apple", "banana", "cherry"]
thislist.clear()
print(thislist)
```

可以直接用 `for` 来遍历。

```python
thislist = ["apple", "banana", "cherry"]
for x in thislist:
  print(x)
```

也可以用下标遍历。

```python
thislist = ["apple", "banana", "cherry"]
for i in range(len(thislist)):
  print(thislist[i])
```

为了性能，也可以用 `while` 来遍历，避免 `range` 生成过大的数组。

```python
thislist = ["apple", "banana", "cherry"]
i = 0
while i < len(thislist):
  print(thislist[i])
  i = i + 1
```

缩写的 `for` 遍历，两边中括号是必须的。

```python
thislist = ["apple", "banana", "cherry"]
[print(x) for x in thislist]
```

赋值的时候，也有一些神奇的语法糖：

```python
fruits = ["apple", "banana", "cherry", "kiwi", "mango"]
newlist = []

newlist = [x for x in fruits if "a" in x]

#Equals to

for x in fruits:
  if "a" in x:
    newlist.append(x)
    
print(newlist)
```

更抽象地：

```python
newlist = [expression for item in iterable if condition == True]
```

还是比较灵活的：

```python
newlist = [x.upper() for x in fruits] 
```

支持直接排序。

```python
thislist = ["orange", "mango", "kiwi", "pineapple", "banana"]
thislist.sort()
print(thislist)
```

排序也有一些参数。

```python
thislist = [100, 50, 65, 82, 23]
thislist.sort(reverse = True)
print(thislist)
```

可以自定义估值函数，返回一个对象用于比较？

```python
def myfunc(n):
  return abs(n - 50)

thislist = [100, 50, 65, 82, 23]
thislist.sort(key = myfunc)
print(thislist)
```

还有这样的：

```python
thislist = ["banana", "Orange", "Kiwi", "cherry"]
thislist.sort(key = str.lower) #case insensitive
print(thislist)
```

所以其实排序内部可能是这样的：

```python
if key(a) > key(b):
    a,b = b,a #swap the objects
```

翻转数组：

```python
thislist = ["banana", "Orange", "Kiwi", "cherry"]
thislist.reverse()
print(thislist)
```

直接拷贝只能拷贝到引用，所以有拷贝数组：

```python
thislist = ["apple", "banana", "cherry"]
mylist = thislist.copy()
print(mylist)
```

也可以直接构造：

```python
thislist = ["apple", "banana", "cherry"]
mylist = list(thislist)
print(mylist)
```

合并：

```python
list1 = ["a", "b", "c"]
list2 = [1, 2, 3]

list3 = list1 + list2
print(list3)
```

总结一下内置函数：

- `append`，尾部追加。
- `clear`，清空。
- `copy`，生成副本。
- `count`，数数用的。
- `extend`，连接两个数组。
- `index`，查找第一个满足条件的元素的下标。
- `insert`，插入。
- `pop`，按下标删除。
- `remove`，按值删除。
- `reverse`，翻转。
- `sort`，排序。

### 元组(Tuple)

元组可以看作是不可修改的 `List`.

用圆括号包裹。

```python
thistuple = ("apple", "banana", "cherry")
print(thistuple)
```

与 `List` 不同的是，单元素的元组声明时，必须加一个句号，否则不会识别为元组。

```python
myList = ["list"]
myTuple = ("tuple") #not Tuple!
myRealTuple = ("tuple",) #is Tuple!
print(type(myList))
print(type(myTuple))
print(type(myRealTuple))
```

构造：

```python
thistuple = tuple(("apple", "banana", "cherry")) # note the double round-brackets
print(thistuple)
```

元组是不可变的(**immutable**)，想要修改只能变成 `List`，改完再生成元组。当然这样做效率很低。

```python
x = ("apple", "banana", "cherry")
y = list(x)
y[1] = "kiwi"
x = tuple(y)

print(x)
```

当我们创建元组时，我们将变量填入，这被称为「打包(**packing**)」.

而我们也可以将元组重新解析为变量，这被称为「解包(**unpacking**)」.

```python
fruits = ("apple", "banana", "cherry")
(green, yellow, red) = fruits

print(green)
print(yellow)
print(red)
```

有趣的是，元组不能修改，却能连接，这大概是因为运算过程产生了新的元组而作为返回值。

```python
tuple1 = ("a", "b" , "c")
tuple2 = (1, 2, 3)

tuple3 = tuple1 + tuple2
print(tuple3)

fruits = ("apple", "banana", "cherry")
mytuple = fruits * 2 #interesting multiply <=> mytuple = fruits + fruits + ... times.

print(mytuple)
```

### 集合(Sets)

这个集合是数学意义上的集合，即具有无序性、不重复性、确定性的特性的集合。

用大括号：

```python
thisset = {"apple", "banana", "cherry"}
print(thisset)
```

集合不支持下标访问，只能遍历：

```python
thisset = {"apple", "banana", "cherry"}

for x in thisset:
  print(x)
```

不能修改元素，但可以添加元素。也可以删除再添加来达到修改的效果。

```python
thisset = {"apple", "banana", "cherry"}

thisset.add("orange")

print(thisset)
```

简单的删除 `remove`，如果删除的元素不存在会报错。

```python
thisset = {"apple", "banana", "cherry"}

thisset.remove("banana")

print(thisset)
```

如果不想要报错，可以用 `discard`：

```python
thisset = {"apple", "banana", "cherry"}

thisset.discard("banana")

print(thisset)
```

甚至可以用 `pop`，由于无序性，可能会随机删除一个元素？

```python
thisset = {"apple", "banana", "cherry"}

x = thisset.pop()

print(x)

print(thisset)
```

取并集，也就是合并两个集合，需要使用 `update`，合并后会去重。

```python
thisset = {"apple", "banana", "cherry"}
tropical = {"pineapple", "mango", "papaya"}

thisset.update(tropical)

print(thisset)
```

当然合并不仅限于集合之间。

```python
thisset = {"apple", "banana", "cherry"}
mylist = ["kiwi", "orange"]

thisset.update(mylist)

print(thisset)
```

如果不想影响原集合，只需要返回值，可以用 `union`：

```python
set1 = {"a", "b" , "c"}
set2 = {1, 2, 3}

set3 = set1.union(set2)
print(set3)
```

取交集：

```python
x = {"apple", "banana", "cherry"}
y = {"google", "microsoft", "apple"}

z = x.intersection(y) #like union
x.intersection_update(y) #like update

print(x)
```

还有更有趣的，删去交集，即 $(\mathbb{A} \cup \mathbb{B}) \setminus (\mathbb{A} \cap \mathbb{B})$

```python
x = {"apple", "banana", "cherry"}
y = {"google", "microsoft", "apple"}

z = x.symmetric_difference(y)
x.symmetric_difference_update(y)

print(x)
```

清空和彻底删除：

```python
thisset = {"apple", "banana", "cherry"}
thisset.clear()
print(thisset)
del thisset
print(thisset)
```

### 字典(Dictionary)

类似于 C++ 中的 `map`，键值对。

3.7 以上的 Python 版本中，字典是有序的。有序性、可变性、不重复性。

```python
thisdict = {
  "brand": "Ford",
  "model": "Mustang",
  "year": 1964
}
print(thisdict)
print(thisdict["brand"])
print(thisdict.get("model")) #the same with the former approach
```

有趣的是，值可以是任意数据类型。

```python
thisdict = {
  "brand": "Ford",
  "electric": False,
  "year": 1964,
  "colors": ["red", "white", "blue"]
}
```

获取所有 `key`：

```python
x = thisdict.keys()
```

值得注意的是，这里传出的是一个引用，也就是说是可以动态更新的。但似乎是只读的。

```python
car = {
"brand": "Ford",
"model": "Mustang",
"year": 1964
}

x = car.keys()

print(x) #before the change

car["color"] = "white"

print(x) #after the change
```

`values` 也是一样的：

```python
car = {
"brand": "Ford",
"model": "Mustang",
"year": 1964
}

x = car.values()

print(x) #before the change

car["year"] = 2020

print(x) #after the change
```

还可以直接获取所有键值对 `items`：

```python
car = {
"brand": "Ford",
"model": "Mustang",
"year": 1964
}

x = car.items()

print(x) #before the change

car["year"] = 2020

print(x) #after the change
```

可以查看键是否存在：

```python
thisdict = {
  "brand": "Ford",
  "model": "Mustang",
  "year": 1964
}
if "model" in thisdict:
  print("Yes, 'model' is one of the keys in the thisdict dictionary")
```

可以用 `update` 来更新，支持塞入一个键值对集合：

```python
thisdict = {
    "brand": "Ford",
    "model": "Mustang",
    "year": 1964
}
another = {
    "another": "Intersting",
    "stuff": "Join it"
}

thisdict.update({"year": 2020})
print(thisdict)
thisdict.update(another)
print(thisdict)
```

移除特定键：

```python
thisdict = {
  "brand": "Ford",
  "model": "Mustang",
  "year": 1964
}
thisdict.pop("model")
print(thisdict)
```

移除最后一个键值对：

```python
thisdict = {
  "brand": "Ford",
  "model": "Mustang",
  "year": 1964
}
thisdict.popitem()
print(thisdict)
thisdict.update({"new":"I'm newer"})
print(thisdict)
thisdict.popitem()
print(thisdict)
```

可以用 `del` 关键字：

```python
thisdict = {
  "brand": "Ford",
  "model": "Mustang",
  "year": 1964
}
del thisdict["model"]
print(thisdict)
```

清空：

```python
thisdict = {
  "brand": "Ford",
  "model": "Mustang",
  "year": 1964
}
thisdict.clear()
print(thisdict)
```

遍历所有键名：

```python
for x in thisdict:
  print(x)
```

遍历所有值：

```python
for x in thisdict:
  print(thisdict[x]) #have to search for the value each time executed
```

直接获取集合来遍历：

```python
for x in thisdict.values():
  print(x)
for x in thisdict.keys():
  print(x)
```

遍历键值对：

```python
for x, y in thisdict.items():
  print(x, y)
```

深拷贝：

```python
thisdict = {
  "brand": "Ford",
  "model": "Mustang",
  "year": 1964
}
mydict = thisdict.copy()
print(mydict)
mydict = dict(thisdict)
print(mydict)
```

嵌套：

```python
child1 = {
  "name" : "Emil",
  "year" : 2004
}
child2 = {
  "name" : "Tobias",
  "year" : 2007
}
child3 = {
  "name" : "Linus",
  "year" : 2011
}

myfamily = {
  "child1" : child1,
  "child2" : child2,
  "child3" : child3
}

print(myfamily["child1"]["name"])
```

## 函数(Functions)

函数定义：

```python
def my_function():
  print("Hello from a function")
my_function()
```

参数：

```python
def my_function(fname):
  print(fname + " Refsnes")

my_function("Emil")
my_function("Tobias")
my_function("Linus")
```

形参(**Parameter**)和实参(**Argument**).

不定长参数：

```python
def my_function(*kids):
  print("The youngest child is " + kids[2])

my_function("Emil", "Tobias", "Linus")
```

可以用更优雅的方式传参：

```python
def my_function(child3, child2, child1):
  print("The youngest child is " + child3)

my_function(child1 = "Emil", child2 = "Tobias", child3 = "Linus")
```

实质上是键值对的传递，因此：

```python
def my_function(**kid):
  print("His last name is " + kid["lname"])

my_function(fname = "Tobias", lname = "Refsnes")
```

默认参数：

```python
def my_function(country = "Norway"):
  print("I am from " + country)

my_function("Sweden")
my_function("India")
my_function()
my_function("Brazil")
```

弱类型，啥都能传：

```python
def my_function(food):
  for x in food:
    print(x)

fruits = ["apple", "banana", "cherry"]

my_function(fruits)
```

返回值：

```python
def my_function(x):
  return 5 * x
```

占位符：

```python
def myfunction():
  pass
```

## Lambda 表达式

只能有一行表达式，但可以有任意个数参数。

```python
lambda arguments : expression
```

例如一个自增 $10$ 的函数：

```python
x = lambda a : a + 10
print(x(5))
```

多参数：

```python
x = lambda a, b, c : a + b + c
print(x(5, 6, 2))
```

有趣的是，可以利用 Lambda 表达式构造匿名函数：

```python
def myfunc(n):
  return lambda a : a * n

mydoubler = myfunc(2)
mytripler = myfunc(3)

print(mydoubler(11))
print(mytripler(11))
```

## 类和对象(Classes/Objects)

Python 是一个面向对象(Object Oriented)的语言。

```python
class MyClass:
  x = 5

p1 = MyClass()
print(p1.x)
```

初始化：

```python
class Person:
  def __init__(self, name, age):
    self.name = name
    self.age = age

p1 = Person("John", 36)

print(p1.name)
print(p1.age)
```

声明方法：

```python
class Person:
  def __init__(self, name, age):
    self.name = name
    self.age = age

  def myfunc(self):
    print("Hello my name is " + self.name)

p1 = Person("John", 36)
p1.myfunc()
```

函数的第一个参数将会是指向自己的引用，并不强制命名为 `self`.

```python
class Person:
  def __init__(mysillyobject, name, age):
    mysillyobject.name = name
    mysillyobject.age = age

  def myfunc(abc):
    print("Hello my name is " + abc.name)

p1 = Person("John", 36)
p1.myfunc()
```

可以删除某个属性：

```python
del p1.age
```

可以删除对象：

```python
del p1
```

占位符：

```python
class Person:
  pass
```

## 继承(Inheritance)

```python
class Person:
  def __init__(self, fname, lname):
    self.firstname = fname
    self.lastname = lname

  def printname(self):
    print(self.firstname, self.lastname)

#Use the Person class to create an object, and then execute the printname method:


x = Person("John", "Doe")
x.printname()


class Student(Person):
  def __init__(self, fname, lname, year):  # overwrite parent's __init__
    super().__init__(fname, lname)
    # <=> Person.__init__(self, fname, lname)
    self.graduationyear = year

  def welcome(self):
    print("Welcome", self.firstname, self.lastname,
          "to the class of", self.graduationyear)

  def printname(self):
      super().printname()
      print("plus {} is a student!".format(self.lastname))

x = Student("Mike", "Olsen", 2020)
x.welcome()
x.printname()
```

## 迭代器(Iterators)

一个迭代器需要有 `__iter__` 和 `__next__` 两个方法。

所有的集合都能提供迭代器，都是可遍历的(**Iterable Containers**).

```python
mytuple = ("apple", "banana", "cherry")
myit = iter(mytuple)

print(next(myit))
print(next(myit))
print(next(myit))
```

创建一个迭代器：

```python
class MyNumbers:
  def __iter__(self):
    self.a = 1
    return self

  def __next__(self):
    if self.a <= 20:
      x = self.a
      self.a += 1
      return x
    else:
      raise StopIteration #Stop iterating

myclass = MyNumbers()
myiter = iter(myclass)

for x in myiter:
  print(x)
```

## 定义域(Scope)

函数中声明的变量只在函数中有效。

```python
def myfunc():
  x = 300
  print(x)

myfunc()
```

事实上，它在该函数的域内有效。

```python
def myfunc():
  x = 300
  def myinnerfunc():
    print(x)
  myinnerfunc()

myfunc()
```

全局变量：

```python
x = 300

def myfunc():
  print(x)

myfunc()

print(x)
```

更多有关全局变量的前文已经说过，这里复习一下。

```python
x = 300

def myfunc():
  global x
  x = 200

def myfunc2():
  x = 400
  print(x)
    
myfunc()
myfunc2()

print(x)
```

## 模块(Modules)

调库大法好。

举个例子，在 `mymodule.py` 中保存以下内容：

```python
person1 = {
  "name": "John",
  "age": 36,
  "country": "Norway"
}
def greeting(name):
  print("Hello, " + name)
```

然后在 `main.py` 中运行：

```python
import mymodule

mymodule.greeting("Jonathan")
a = mymodule.person1["age"]
print(a)
```

可以起别名(**Alias**)：

```python
import mymodule as mx

a = mx.person1["age"]
print(a)
```

有一些内置的模块：

```python
import platform

x = platform.system()
print(x)
x = dir(platform)
print(x)
```

可以指定引用模块的某些部分，此时不需要再写模块名：

```python
from mymodule import person1
print (person1["age"])
#print(mymodule.person1["age"]) WRONG!!
```

也可以起别名：

```python
from mymodule import person1 as p1
print (p1["age"])
```

## PIP

包管理器。

安装包：`pip install <package-name>`
例如：`pip install camelcase`

然后就能直接使用了：

```python
import camelcase

c = camelcase.CamelCase()

txt = "hello world"

print(c.hump(txt))
```

## 异常捕获(Try...Except)

比较常规。

```python
try:
  print(x)
except NameError:
  print("Variable x is not defined")
except:
  print("Something else went wrong")
else:
  print("Nothing went wrong")
finally:
  print("Ended.")
```

举个例子：

```python
try:
  f = open("demofile.txt")
  f.write("Lorum Ipsum")
except:
  print("Something went wrong when writing to the file")
finally:
  f.close()
```

抛出异常：

```python
x = -1

if x < 0:
  raise Exception("Sorry, no numbers below zero")
```

可以指定类型：

```python
x = "hello"

if not type(x) is int:
  raise TypeError("Only integers are allowed")
```

## 输入(Input)

很简单的输入。

```python
username = input("Enter username:")
print("Username is: " + username)
```

## 格式化字符串(Formatting)

前文已经简单提及了。

```python
price = 49
txt = "The price is {} dollars"
print(txt.format(price))
```

可以指定输出格式：

```python
quantity = 3
itemno = 567
price = 49
myorder = "I want {0} pieces of item number {1} for {2:.2f} dollars."
print(myorder.format(quantity, itemno, price))
```

可以重复利用：

```python
age = 36
name = "John"
txt = "His name is {1}. {1} is {0} years old."
print(txt.format(age, name))
```

可以传键值对：

```python
myorder = "I have a {carname}, it is a {model}."
print(myorder.format(carname = "Ford", model = "Mustang"))
```

## 结语

差不多把 Python 中的基础语法过了一遍，相信各位读者读完后都能入门吧。

大部分编程概念都是相似的，学习起来并不困难。这也是一个写起来没什么心智负担的工具语言。有什么需要直接面向谷歌编程即可。