# 快速开始<div id="introduce"></div>

该文档简要介绍了peewee的主要功能。本指南将涵盖：

* [模型定义（Model Definition）]()
* [存储数据（Storing data）]()
* [检索数据（Retrieving Data）]()

> 如果你想要更加丰富多彩的东西，可以使用peewee和flask框架创建一个"twitter"风格的web应用程序。在项目examples/文件夹中，可以找到更多自包含的peewee例子，例如博客应用程序。

强烈建议打开交互式shell会话并运行代码，通过这种方式，可以感受到输入查询。


# 模型定义（Model Definition）<div id="model"></div>

模型类，字段和模型实例都映射到数据库概念：

Object | Corresponds to...
--|--
Model class | Database table
File instance | Column on a tabel
Model install | Row in a database table

使用peewee启动项目时，通常最好从数据模型开始，通过定义一个或多个Model类：、


```python
# 创建了Person表
from peewee import *
db = SqliteDatase('people.db')

class Person(Model):
  name = CharField()
  birthday = DateField()

  class Meta:
    database = db # 该模型使用‘people.db’数据库
```

> Peewee将自动从类的名称推断出数据库表名。你可以通过内部“meta” 类中指定table_name属性（以及数据库属性）来覆盖默认名称。要了解有关Peewee如何生成表名更多信息，请参阅表名称部分。
另请注意，我们将模型命名为`Person`而不是`People`。这是一个你应该遵循的约定-即使表包含多个人，我们总是使用单数形式命名该类。

有许多是和存储各类型数据的字段类型。Peewee处理数据库使用的pythonic值之间的转换，因此你可以在代码中使用python类型而不必担心。

> 使用外键关系建立模型之间的关系时，peewee这样子解决：

```python
# 创建了 Pet 表
class Pet(Model):
  owner = ForeignKeyField(Person, backref='pets')
  name = CharField()
  animal_type = CharField()

  class Meta:
    database = db; # 该模型使用‘people.db’数据库
```

想在我们有了模型，让我们连接到数据库。虽然没有必要明确打开连接，但这是一种很好的做法，因为它会立即显示数据库连接的任何错误，而不是在执行第一个查询后的某个任意时间。完成后关闭连接也很好- 例如，web应用程序可能在收到请求时打开连接，并在发送响应时关闭连接

```python
db.connect()
```
我们首先在数据库中创建将存储数据的表。这将创建具有适当的列，索引，序列，和外键约束的表：

```python
db.create_tables([Person, Pet])
```

# 存储数据(Storing data)

让我们从一些人填充数据库开始。我们将使用`save()`和 `create()`方法来添加和更新人民的记录。

```python
#  创建 uncle_bob
from datetime import date
uncle_bob = Person(name='Bob', birthday=date(1960,1,15))
uncle_bob.save() # bob现在存储在数据库中了
#returns: 1

```

> 调用`save()`时，将返回修改的行数。

你还可以通过调用`create()`方法添加一个人，该方法返回 一个模型实例：

```python
# 创建 Grandma , Herb
grandma = Person.create(name='Grandma', birthday=date(1935, 3,1))
herb = Person.create(name='Herb', birthday=date(1950,5,5))
```

要更新行，请修改模型实例并调用`save()`以保留更改。在这里，我们将更改Grandma的名称，然后将更改保存在数据库中：

```python
grandma.name = 'Grandma L.'
grandma.save() # 更新数据库中Grandma的名称
# returns: 1 (返回更改成功的行数)
```

现在已存储了3个人的数据。给他们添加一些宠物。Grandma 不喜欢宠物，她不会用任何动物，但是Herb是动物爱好者。

```python
bob_kitty = Pet.create(owner=uncle_bob, name='kitty', animal_type='cat')
herb_fido = Pet.create(owner=herb, name='Fido', animal_type='dog')
herb_mittens = Pet.create(owner=herb, name='Mittens', animal_type='cat')
herb_mittens_jr = Pet.create(owner=herb, name='Mittens Jr', animal_type='cat')

```

但是经过一段时间后，'Mittens'病死，将它从数据库中删除

```python
herb_mittens.delete_instance()
# Retauns: 1
```

>  delete_instance()返回值是从数据库中删除的行数。

uncle Bob从Herb中获取其中一个动物
```python
herb_fido.owner = uncle_bob
herb_fido.save()

```

# 检索数据（Retrieving Data）

我们数据库真正优势在于它如何允许我们通过查询检索数据。关系数据库非常适合进行ad-hoc queries(即席查询)

## 获取单一记录

从数据库中检索Grandma记录。从数据库中获取单个记录，使用`Select.get()`

```python
grandma = Person.select().where(Person.name == 'Grandma L.').get()

```

我们也可以使用等效的简写`Model.get()`:
```python
grandma = Person.get(Pserson.name == 'Grandma L.')
```

## 记录清单

列出数据库所有人：
```python
for person in Person.select():
  print(person.name)
# Bob
# Grandma L.
# Herb
```

列出所有猫以及其主人的名字：
```python
query = Pet.select().where(Pet.animal_type == 'cat')
for pet in query:
  print(pet.name, pet.owner.name)

# kitty Bob
# Mittens Jr Herb
```

> 上一个查询存在一个很大的问题：因为我们正在访问pet.owner.name,而我们没有在原始查询中选择彼此关系，所以peewee必须执行额外的查询来检索宠物所有者。此行为称为N+1，通常避免使用此行为。
有关使用关系和联接的深入指南，参阅“”文档。

通过选择Pet和Person以及添加连接来避免额外的查询。

```python
query = (Pet
          .select(Pet, Person)
          .join(Person)
          .where(Pet.animal_type == 'cat'))
for pet in query:
  print(pet.name, pet.owner.name)
```

获取Bob的所有宠物

```python
for pet in Pet.select().join(Person).where(Person.name == 'Bob'):
  print(pet.name)

```

使用另一种更好的方法获取Bob的宠物。
```python
for pet in Pet.select().where(Pet.owner == uncle_bob):
  print(pet.name)

```

## 排序（Sorting）

通过添加order_by()子句按字母顺序对它们进行排序
```python
for pet in Pet.select().where(Pet.owner == uncle_bob).order_by(Pet.name)

```

按年龄的降序列出所有人, 从年轻到年老
```python
for person in Person.select().order_by(Person.birthday.desc()):
  print(person.name, person.birthday)

# Bob 1960-01-15
# Herb 1950-05-05
# Grandma L. 1935-03-01
```

## 组合过滤器表达式（Combining filter expressions）

Peewee 支持任意嵌套的表达式。让所有生日都是以下的人：

* before 1940(grandma)
* after 1959(bob)

```python
d1940 = date(1940, 1, 1)
d1960 = date(1960, 1, 1)
query = (Person
          .select()
          .where((Person.birthday < d1940) | (Person.birthday > d1960)))
for person jin query:
  print(person.name, person.birthday)

# Bob 1960-01-15
# Grandma L. 1935-03-01
```

查询生日在1940年到1960年之间的人：

```python 
query = (Person.
          .select().
          where(Person.birthday.between(d1940, d1960)))
for person in query:
  print(person.name, person.birthday)

# Herb 1950-05-05

```

## 聚合和预取（Aggregates and Prefetch）

列出所有人和他们宠物数量：
```python
for person in Person.select():
  print(person.name, person.pets.count(), 'pets')

# Bob 2 pets
# Grandma L. 0 pets
# Herb 1 pets
```

再一次遇到了N+1查询行为的经典示例。在这种情况下，我们正在为原始`select`返回的每个Person执行一个额外的查询。我们可以通过执行`join`并使用SQL函数来聚合结果来避免这种情况。

```python
query = (Person
          .select(Person, fn.COUNT(Pet.id).alias('pet_count'))
          .join(Pet, JOIN.LEFT_OUTER)
          .group_by(Person)
          .order_by(Person.name))
for person in query:
  print(person.name, person.pet_count, 'pets')

# Bob 2 pets
# Grandma L. 0 pets
# Herb 1 pets
```
> peewee 提供了一个神奇的帮手`fn()`，可用于调用任何SQL函数。在上面的示例中，`fn.COUNT(Pet.id).alias('pet_count')`将被转换为`COUNT(pet.id) as pet_count`








