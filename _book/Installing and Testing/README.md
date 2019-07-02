## 安装和测试<div id="introduce"></div>

大多数用户只想安装PyPi上托管的最新版本：
```python
pip install peewee

```
Peewee带有几个C扩展，如果Cython可用，它将被构建。

* Sqlite扩展，包括SQLite日期操作函数的Cython实现，REGEXP运算符和全文搜索结果排名算法。


## 使用git安装<div id="git"></div>

该项目托管在<https://github.com/coleifer/peewee>，可以使用git安装：

```python
git clone https://github.com/coleifer/peewee.git
cd peewee
python setup.py install

```
> 在某些系统上，可能需要使用 `sudo python setup.py install` 在系统范围内安装peewee。

在git checkout中构建SQLite扩展，可以运行：
```python
# 构建C扩展并将共享库与其他模块放在一起。
python setup.py build_ext -i

```


# 运行测试套件 <div id="test"></div>

通过运行测试套件来测试已安装的。

```python
python runtests.py

```

使用runtests.py脚本测试特定功能或特定数据库驱动。查看可用的测试运行器选项：
```python
python runtests.py --help

```
> 要对Postgres或MySQL运行测试，需要创建一个名为“peewee_test”的数据库。要测试Postgres扩展模块，还需要在postgres测试数据库中安装HStore扩展,运行命令：
```python
CREATE EXTENSION hstore;
```


# 可选的依赖项 <div id="dev"></div>

> 要使用Peewee, 你通常不需要标准库之外的任何内容，因为大多数python发行版都是使用SQLite支持编译的。可以通过在Python控制台中运行`import sqlite3`来进行测试。如果想使用另一个数据库，那里有许多与DB-API 2.0兼容的驱动程序，例如分别用于MYSQL和Postgres的`pymysql`或`psycopg2`。

* Cython: 用于在使用SQLite时公开其他功能，并以高效的方式实现搜索结果排名。由于生成的C文件包含在软件包分发中，因此不再需要Cython来使用C扩展。
* apsw: 可选的第三方SQLite绑定，为SQLite的C APIs提供更高的性能和全面的支持。与`APSWDatabse`一起使用
* [gevent](http://www.gevent.org/)是SqliteQueueDatabase的可选依赖项（虽然它可以很好地处理线程）。
* BerkeleyDB可以与Peewee一起使用的SQLite前端进行编译。（[编译说明](http://charlesleifer.com/blog/updated-instructions-for-compiling-berkeleydb-with-sqlite-for-use-with-python/)）
* 最后，如果你使用Flask框架，可以使用帮助程序扩展模块

# SQL扩展的注意项

Peewee包含两个特定于SQLite的C扩展，为SQLite数据库用户提供了额外的功能和改进的性能。如果安装了SQLite3，Peewee将尝试提前确定，如果您的系统上有SQLite共享库，则只构建SQLite扩展。

但是，如果在尝试安装Peewee时收到如下错误，则可以通过设置NO_SQLITE环境变量显式禁用SQLite C扩展的编译。

```
fatal error: sqlite3.h: No such file or directory

```
以下是如何安装Peewee并明确禁用SQLite扩展：

```
$ NO_SQLITE =1 python setup.py install
```


