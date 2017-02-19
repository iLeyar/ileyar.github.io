---
title: "Ubuntu 下 GoldenDict 词典的安装与使用"
date: 2015-06-21 23:59:59
tags: 
- ubuntu
- goldendict
---

作为一个经常需要接触英语的人来说，一本优质的词典必不可少。而对于每天都会接触手机，电脑的我，使用词典程序是最佳选择。

在 windows 系统下我用的欧陆词典，Adroid 下用的 BlueDict，这两个程序在 Linux 下都没有客户端。Google 后决定使用 GoldenDict。该程序支持的词典格式丰富，例如 dsl、bgl、stardict...

下面来试试这款软件吧。

<!--more-->

GoldenDict 安装
----------------

+ Ubuntu 下安装：` sudo apt-get install goldendict`
+ [github](https://github.com/goldendict/goldendict) 编译
+ 其他详细安装方法：[链接](http://forum.ubuntu.org.cn/viewtopic.php?f=95&t=265588)

搭配词典及使用方法
------------------

### 词典推荐

+ Longman Dictionary of Contemporary English 5th edition (En-En\_Longman\_DOCE5) 朗文当代第五版词典
+ Longman Pronunciation Dictionary 3rd edition (En-En\_Longman\_Pronunciation3) 朗文发声词典第三版
+ Merriam-Webster's Collegiate 11th edition (En-En\_Merriam\_Webster11) 韦式大学词典第11版
+ Oxford Advanced Learner's Dictionary 8th edition (En-En\_OALD8) 牛津高阶词典第8版
+ (En-zh\_CN\_OALD4) 牛津高阶英汉双解

以上五部词典可在这里下载：[Dropbox 分享链接](https://www.dropbox.com/sh/bf1v7wthsl7pmbi/qt3D1kvFmv)

### 词典安装

将词典文件放在任意目录下，通过 Edit -> Dictionaries -> Files -> Add 把目录选择添加进去，然后勾选 Recursive 即可。

### 词形匹配 

使用 Goldendict 取词的时候会出现复数形式的单词查询不到，通过设置构词法规则库即可解决。

下载并解压以下目录，设置 Edit -> Dictionaries -> Morphology 路径为该目录即可。

+ [en\_US\_1.0.zip](https://www.dropbox.com/s/dda9n4sok28wek7/en_US_1.0.zip) 词形匹配，查词时自动将复数或者其他形式转换为标准形式

### 词典文件目录树

```
goldendict
├── En-En_Longman_DOCE5
│   ├── En-En-Longman_DOCE5.ann
│   ├── En-En-Longman_DOCE5.bmp
│   ├── En-En-Longman_DOCE5.dsl.dz
│   ├── En-En-Longman_DOCE5.dsl.files.zip
│   ├── En-En-Longman_DOCE5_Extras.ann
│   ├── En-En-Longman_DOCE5_Extras.bmp
│   └── En-En-Longman_DOCE5_Extras.dsl.dz
├── En-En_Longman_Pronunciation3
│   ├── En-En-Longman_Pronunciation_abrv.dsl.dz
│   ├── En-En-Longman_Pronunciation.ann
│   ├── En-En-Longman_Pronunciation.bmp
│   ├── En-En-Longman_Pronunciation.dsl.dz
│   └── En-En-Longman_Pronunciation.dsl.dz.files.zip
├── En-En_Merriam_Webster11
│   ├── En-En-MWCollegiate11.ann
│   ├── En-En-MWCollegiate11.bmp
│   ├── En-En-MWCollegiate11.dsl.dz
│   └── En-En-MWCollegiate11.dsl.dz.files.zip
├── En-En_OALD8
│   ├── En-En_Oxford Advanced Learners Dictionary.ann
│   ├── En-En_Oxford Advanced Learners Dictionary.bmp
│   ├── En-En_Oxford Advanced Learners Dictionary.dsl.dz
│   └── En-En_Oxford Advanced Learners Dictionary.dsl.dz.files.zip
├── en_US_1.0
│   ├── en_US.aff
│   ├── en_US.dic
│   └── en_US.txt
└── En-zh_CN_OALD4
    └── Oxford_Advanced_Learner_English-Chinese_Dictionary-4th.bgl
```

其他
-----------

安装一切都顺利，暂时未遇到其他问题。Enjoy~
