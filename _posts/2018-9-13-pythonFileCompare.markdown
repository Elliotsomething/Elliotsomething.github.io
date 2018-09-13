---
layout:     post
title:      "python 两个文件词条对比"
subtitle:   "Python File Compare"
date:       2018-9-13
author:     "Elliot"
header-img: "img/post-bg-ios9-web.jpg"
catalog:    true
tags:
    - Python
---

**版权声明：本文为博主原创文章，未经博主允许不得转载**


两个文件对比 词条的脚本，比如两个文件cn.strings和tw.strings，会把不同的词条找出来，并且删除输出到一个文件里

cn.strings:
```c

"voov_safety_admin_list_footer"="最多可以添加5位管理員，管理員可以協助封鎖沒禮貌觀眾的發言或將他們移出直播室";
"voov_safety_admin_list_no_admin"="暫無管理員";

Aaaaaaaaaa
Bbbbbbbbbbbbbbb

"voov_jb_des_content"="金幣可用於在 JOOX V中購買禮物贈送給喜愛的主播和用於發送彈幕訊息"
```

tw.strings:
```c
"voov_safety_admin_list_footer"="最多可以添加5位管理員，管理員可以協助封鎖沒禮貌觀眾的發言或將他們移出直播室";
"voov_safety_admin_list_no_admin"="暫無管理員";


"voov_jb_des_content"="金幣可用於在 JOOX V中購買禮物贈送給喜愛的主播和用於發送彈幕訊息"
```

```c
# -*- coding: UTF-8 -*-
#python 3.x
#python 对比两个string文件内容

import os

def main():
    readFile('test1.txt','test2.txt')
    # readFile('Localizable_zh_CN.strings')

def readFile(filename1,filename2):
    with open(filename1) as file1,\
        open(filename2) as file2:
        fa = file1.readlines()
        fb = file2.readlines()
        file1.close()
        file2.close()
    with open(filename1,'w') as wfile1:
        for m in fa:
            m = m.rstrip('\n')
            strlist = m.split('=')
            line = strlist[0]
            for n in fb:
                flag = -1
                if line in n:
                    flag = 1
                    wfile1.write(m + '\n')
                    break
            if flag == -1:
                file = open(filename1 + 'new' + '.txt','r+')
                file.seek(0,0)
                file.read()
                file.writelines(m + '\n')
                file.close()
                print(m)
        wfile1.close()

if __name__ == "__main__":
    main()
```
