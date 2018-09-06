---
layout:     post
title:      "python初步学习"
subtitle:   "python Learning"
date:       2018-9-6
author:     "Elliot"
header-img: "img/post-bg-ios9-web.jpg"
catalog:    true
tags:
    - python
---

**版权声明：本文皆摘抄自网络，仅用于学习参考；如有侵权，请随时联系。**

#### python初步学习

##### 创建文件夹

```c
import os

def mkdir(path):

    folder = os.path.exists(path)

    if not folder:                   #判断是否存在文件夹如果不存在则创建为文件夹
        os.makedirs(path)            #makedirs 创建文件时如果路径不存在会创建这个路径
        print "---  new folder...  ---"
        print "---  OK  ---"

    else:
        print "---  There is this folder!  ---"

file = "G:\\xxoo\\test"
```

os.getcwd()可以查看py文件所在路径；
在os.getcwd()后边 加上 [:-4] + 'xxoo\\' 就可以在py文件所在路径下创建 xxoo文件夹

```c
import os

folder = os.getcwd()[:-4] + 'new_folder\\test\\'
#获取此py文件路径，在此路径选创建在new_folder文件夹中的test文件夹

if not os.path.exists(folder):
    os.makedirs(folder)
```

###### 创建txt文件
在桌面创建一个名字为 new 的txt文件

```c
import os

file = open('C:\\Users\Administrator\\Desktop\\' + 'new' + '.txt','w')
file.close()
```

在py文件路径下创建test的txt文件

```c
import os

def txt(name,text):              #定义函数名
    b = os.getcwd()[:-4] + 'new\\'

    if not os.path.exists(b):     #判断当前路径是否存在，没有则创建new文件夹
        os.makedirs(b)

    xxoo = b + name + '.txt'    #在当前py文件所在路径下的new文件中创建txt

    file = open(xxoo,'w')

    file.write(text)        #写入内容信息

    file.close()
    print ('ok')
txt('test','hello,python')       #创建名称为test的txt文件，内容为hello,python
```

##### python 文件读写查找、替换相关简单操作

python进行文件读写的函数是open或file
`file_handler = open(filename,,mode）`

###### file的读写方法：

F.read([size]) #size为读取的长度，以byte为单位

F.readline([size])

- 读一行，如果定义了size，有可能返回的只是一行的一部分

F.readlines([size])

- 把文件每一行作为一个list的一个成员，并返回这个list。其实它的内部是通过循环调用readline()来实现的。如果提供size参数，size是表示读取内容的总长，也就是说可能只读到文件的一部分。

F.write(str)

- 把str写到文件中，write()并不会在str后加上一个换行符

F.writelines(seq)

- 把seq的内容全部写到文件中。这个函数也只是忠实地写入，不会在每行后面加上任何东西。

###### file的其他方法：


F.close()

- 关闭文件。python会在一个文件不用后自动关闭文件，不过这一功能没有保证，最好还是养成自己关闭的习惯。如果一个文件在关闭后还对其进行操作会产生ValueError

F.flush()

- 把缓冲区的内容写入硬盘

F.fileno()

- 返回一个长整型的”文件标签“

F.isatty()

- 文件是否是一个终端设备文件（unix系统中的）

F.tell()

- 返回文件操作标记的当前位置，以文件的开头为原点

F.next()

- 返回下一行，并将文件操作标记位移到下一行。把一个file用于for ... in file这样的语句时，就是调用next()函数来实现遍历的。

F.seek(offset[,whence])

- 将文件打操作标记移到offset的位置。这个offset一般是相对于文件的开头来计算的，一般为正数。但如果提供了whence参数就不一定了，whence可以为0表示从头开始计算，1表示以当前位置为原点计算。2表示以文件末尾为原点进行计算。需要注意，如果文件以a或a+的模式打开，每次进行写操作时，文件操作标记会自动返回到文件末尾。

F.truncate([size])

- 把文件裁成规定的大小，默认的是裁到当前文件操作标记的位置。如果size比文件的大小还要大，依据系统的不同可能是不改变文件，也可能是用0把文件补到相应的大小，也可能是以一些随机的内容加上去。


##### python--文件操作删除某行

第一种：是先把文件读入内存，在内存中修改后再写入源文件。
例子：将内容包含“123”的所有行删去：

```c
with open('C:/Users/lai/Desktop/1.txt','r') as r:
    lines=r.readlines()
with open('C:/Users/lai/Desktop/1.txt','w') as w:
    for l in lines:
       if '123' not in l:
          w.write(l)
```

第二种：我们可以使用 open() 方法把需要修改的文件打开为两个文件，然后逐行读入内存，找到需要删除的行时，用后面的行逐一覆盖。实现方式见以下代码。

```c
with open('file.txt', 'r') as old_file:
  with open('file.txt', 'r+') as new_file:

    current_line = 0

    # 定位到需要删除的行
    while current_line < (del_line - 1):
      old_file.readline()
      current_line += 1

    # 当前光标在被删除行的行首，记录该位置
    seek_point = old_file.tell()

    # 设置光标位置
    new_file.seek(seek_point, 0)

    # 读需要删除的行，光标移到下一行行首
    old_file.readline()

    # 被删除行的下一行读给 next_line
    next_line = old_file.readline()

    # 连续覆盖剩余行，后面所有行上移一行
    while next_line:
      new_file.write(next_line)
      next_line = old_file.readline()

    # 写完最后一行后截断文件，因为删除操作，文件整体少了一行，原文件最后一行需要去掉
    new_file.truncate()
```
