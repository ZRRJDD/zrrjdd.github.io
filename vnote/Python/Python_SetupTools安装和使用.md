# Python_SetupTools安装和使用

 [Python包管理工具setuptools详解](https://www.jianshu.com/p/ea9973091fdf)

## 1.什么是SetupTools

SetupTools是Python `distutils`增强版的集合，它可以帮助我们更简单的创建和分发Python包，尤其是拥有依赖关系的。用户再使用setuptools创建的包时，并不需要已安装setuptools，只要启动模块即可。

### 优点

- 利用EasyInstall自动查找、下载、安装、升级依赖包
- 创建Python Eggs
- 包含包目录内的数据文件
- 自动包含包目录内的所有的包，而不用在setup.py中列举
- 自动包含包内核发布有关的所有相关文件，而不用创建一个MANIFEST.in文件
- 自动生成经过包装的脚本或Windows执行文件
- 支持Pyrex，即在可以setup.py中列出.pyx文件，而最终用户无序安装Pyrex
- 支持上传到PyPI
- 可以部署开发模式，使项目在sys.path中
- 用新命令或setup()参数扩展distuils，为多个项目发布/重用扩展
- 在项目setup()中简单声明entry points，创建可以自动发展扩展的应用和框架。

**总之，setuptools就是比distutils好用的多，基本满足大型项目的安装和发布**

## 2.安装setuptools
### 2.1 最简单安装，假定在ubuntu下
```shell
sudo apt-get install python-setuptools
```
### 2.2 启动脚本安装
```shell
wget http://peak.telecommunity.com/dist/ez_setup.py
sudo python ez_setup.py
```

## 3.创建一个简单的包
有了setuptools后，创建一个包基本上是无脑操作。
```shell
cd /tmp
mkdir demo
```
在demo中创建一个setup.py文件，写入
```python
from setuptools import setup,find_packages
setup(
    name = "demo",
    version = "0.1",
    packages = find_packages()
```

执行`python setup.py bdist_egg` 即可打包一个test的包了。
```
demo
├── build
│   └── bdist.macosx-10.7-x86_64
├── demo.egg-info
│   ├── PKG-INFO
│   ├── SOURCES.txt
│   ├── dependency_links.txt
│   └── top_level.txt
├── dist
│   └── demo-0.1-py3.7.egg
└── setup.py
```
在dist中生成的egg包
```
(base) zrrjdd@zrrjdddeMacBook-Pro demo % file dist/demo-0.1-py3.7.egg 
dist/demo-0.1-py3.7.egg: Zip archive data, at least v2.0 to extract
```
看一下生成的.egg文件，是个zip包，解开看看先
```
unzip -l dist/demo-0.1-py3.7.egg

Archive:  dist/demo-0.1-py3.7.egg
  Length      Date    Time    Name
---------  ---------- -----   ----
      176  01-27-2020 13:32   EGG-INFO/PKG-INFO
      120  01-27-2020 13:32   EGG-INFO/SOURCES.txt
        1  01-27-2020 13:32   EGG-INFO/dependency_links.txt
        1  01-27-2020 13:32   EGG-INFO/top_level.txt
        1  01-27-2020 13:32   EGG-INFO/zip-safe
---------                     -------
      299                     5 files
```

我们可以看到，里面是一系列自动生成的文件，现在可以介绍一下setup()中的参数了
```
name：包名
version：版本号
packages 所包含的其他包
```
要想发不到PyPI中，需要增加别的参数，可以参考官方文档中的例子了。

## 4.给包增加内容
上面生成的egg中没有实质的内容，显然谁也用不了，现在我们稍微调整一下，增加一点内容。

在demo中执行mkdir demo，再创建一个目录，在这个demo目录中创建一个`init.py`的文件，表示这个目录是一个包，然后写入
```python
#!/usr/bin/env python
#-*- coding:utf-8 -*-

def test():
    print('hello world!')

if __name__ == '__main__':
    test()
```
现在的主目录结构如下：
```
├── build
│   └── bdist.macosx-10.7-x86_64
├── demo
│   └── __init__.py
├── demo.egg-info
│   ├── PKG-INFO
│   ├── SOURCES.txt
│   ├── dependency_links.txt
│   └── top_level.txt
├── dist
│   └── demo-0.1-py3.7.egg
└── setup.py
```

再次执行`python setup.py bdist_egg`后，再看egg包
```
Archive:  dist/demo-0.1-py3.7.egg
  Length      Date    Time    Name
---------  ---------- -----   ----
      176  01-27-2020 14:12   EGG-INFO/PKG-INFO
      137  01-27-2020 14:12   EGG-INFO/SOURCES.txt
        1  01-27-2020 14:12   EGG-INFO/dependency_links.txt
        5  01-27-2020 14:12   EGG-INFO/top_level.txt
        1  01-27-2020 14:12   EGG-INFO/zip-safe
      122  01-27-2020 14:10   demo/__init__.py
      292  01-27-2020 14:12   demo/__pycache__/__init__.cpython-37.pyc
---------                     -------
      734                     7 files
```

这回包内多了demo目录，显然已经有了我们自己的东西，安装体验一下。
```shell
python setup.py install
```
这个命令会将我们创建的egg安装到python的dist-packages目录下，我这里的位置在 `/Users/zrrjdd/anaconda3/lib/python3.7/site-packages/demo-0.1-py3.7.egg`

查看一下它的结构：
```
/usr/local/lib/python2.7/dist-packages/demo-0.1-py2.7.egg
|-- demo
|   |-- __init__.py
|   `-- __init__.pyc
`-- EGG-INFO
    |-- dependency_links.txt
    |-- PKG-INFO
    |-- SOURCES.txt
    |-- top_level.txt
    `-- zip-safe
```
打开python终端或者ipython 都行，直接导入我们的包
```ipython
>>> import demo
>>> demo.test()
hello world!
>>> 
```
好的，执行成功。

## 5.setuptools进阶

在上例子中，我们基本都是使用setup()的默认参数，这只能写一些简单的egg。一旦我们的project逐渐变大以后，维护起来就有点复杂了。下面是setup()的其他参数，可以学习一下：

### 5.1 find_packages()
对于简单工程来说，手动增加packages参数很容易，刚刚我们用到了这个函数，他默认在和setup.py同一目录下搜索各个含有init.py的包。其实我们可以将包统一放到一个src目录下，另外包内可能还有aaa.txt文件和data数据文件夹。

```
demo
├── setup.py
└── src
    └── demo
        ├── __init__.py
        ├── aaa.txt
        └── data
            ├── abc.dat
            └── abcd.dat
```
如果不加控制，则setuptools只会将init.py加入到egg中，想要将这些文件都添加，需要修改setup.py
```python
from setuptools import setup,find_packages
setup(
    packages = find_packages('src'), #包含所有src中的包
    package_dir = {'':'src'}, #告诉distutils包都在src下

    package_data = {
        #任何包中含有.txt文件，都含它
        '':['*.txt'],
        #包含demo包data文件夹中的 *.dat文件
        'demo':['data/*.dat']
    }
)
```
另外，也可以排除一些特定的包，如果在src中再添加一个tests包，可以通过exclude来排除。
```python
find_packages(exclude["*.tests","*.tests.*","tests.*","tests"])
```

### 5.2 entry_points
一个字典，从entry_point组名映射到一个表示entry_point的字符串或字符串列表。Entry_points是用来支持动态发现服务和插件，也用来支持自动生成脚本。这个还是看例子比较好理解：
```python
setup(
    entry_points = {
        'console_scripts':[
            'foo = demo:test',
            'bar = demo:test'
        ],
        'gui_scripts':[
            'baz = demo:test'
        ]
    }
)
```

修改setup.py增加以上内容以后，再次安装这个egg，可以发现在安装信息里头多了两行代码：
```
Installing foo script to /usr/local/bin
Installing bar script to /usr/local/bin
```

查看`/usr/local/bin/foo`内容
```shell
#!/usr/bin/python
# EASY-INSTALL-ENTRY-SCRIPT: 'demo==0.1','console_scripts','foo'
__requires__ = 'demo==0.1'
import sys
from pkg_resources import load_entry_point

if __name__ == '__main__':
    sys.exit(
        load_entry_point('demo==0.1', 'console_scripts', 'foo')()
    )
    
```
这个内容其实显示的意思是，foo将执行console_scripts定义的foo所代表的函数。执行foo，发现打出了hello world！，和预期结果一样。

- 使用Eggsecutable Scripts
- 从字面上理解这个词，Eggsecutable是Eggs和executable合成词，翻译过来就是另eggs可执行。也就是说定义好一个参数以后，可以让你生成.egg文件可以被直接执行，貌似Java的.jar也有这机制。下面是使用方法：

```python
setup(
    #other arguments here ...
    entry_points = {
        'setuptools.installation':[
            'eggsecutable = demo:test'
        ]
    }
)
```

这么写意味着在执行python *.egg时，会执行我的test()函数，在文档中说需要将.egg放到PATH路径中。

### 5.3 包含数据文件

在3中我们已经列举了如何包含数据文件，其实setuptools提供的不只这么一种方法，下面是另外两种。

#### 5.3.1 包含所有包内文件

这种方法中包含所有文件指的是受版本控制（CVS/SVN/GIT等）的文件，或者通过MANIFEST.in声明的。
```python
from setuptools import setup,find_packages
setup(
    ...
    include_package_data = True
)
```

#### 5.3.2 包含一部分，排除一部分
```python
from setuptools import setup,find_packages
setup(
    ...
    packages = find_packages('src'),
    package_dir = {'':'src'},

    include_package_data = True,
    #排除所有 README.txt
    exclude_package_data = {'':['README.txt']}
)
```

如果没有使用版本控制的话，可以还是使用3中提到的包含方法
### 5.4 可扩展的框架和应用

setuptools可以帮助你将应用变成插件模式，供别的应用使用。官网举例是一个帮助博客更改输出类型的插件，一个博客可能想要输出不同类型的文章，但是总自己写输出格式化代码太繁琐，可以借助一个已经写好的应用，在编写博客程序的时候动态调用其中的代码。

通过entry_points可以定义一系列接口，供别的应用或者自己调用。例如
```python
setup(
    entry_points = {'blogtool.parsers':'.rst = some_module:SomeClass'}
)

setup(
    entry_points = {'blogtool.parsers':['.rst = some_module:a_func']}
)

setup(
    entry_points = """
        [blogtool.parsers]
        .rst = some.nested.module:SomeClass.some_classmethod[reST]
    """,
    extras_require = dict(reST = "Docutils>=0.3.5")
)

```
上面列举了三种定义方式，即我们将我们some_module中的函数，以名字为blogtool.parsers的借口共享给别的应用。

别的应用使用的方法是通过pkg_resources.require()来导入这些模块。

另外，一个名叫stevedore的库将这个方式做了封装，更加方便进行应用的扩展。



