---
title: Python-Template 使用指南
date: 2020-11-2
tags: Python-Template
---

Template 无疑是一个好东西，可以将字符串的格式固定下来，重复利用。同时 Template 也可以让开发人员可以分别考虑字符串的格式和其内容了，无形中减轻了开发人员的压力。

Template 属于 string 中的一个类，所以要使用的话可以用以下方式调用

```
from string import Template
```

Template 有个特殊标示符 $, 它具有以下的规则：

它的主要实现方式为 $xxx, 其中 xxx 是满足 python 命名规则的字符串，即不能以数字开头，不能为关键字等

如果 $xxx 需要和其他字符串接触时，可用 {} 将 xxx 包裹起来（以前似乎使用'()’, 我的一本参考书上是这样写的，但是现在的版本应该只能使用'{}’）。例如，aaa${xxx}aaa

Template 中有两个重要的方法：substitute 和 safe_substitute.

这两个方法都可以通过获取参数返回字符串

```python
>>s=Template(There $a and $b)
>>print(s.subtitute(a='apple',b='banana'))
There apple and banana
>>print(s.safe_substitute(a='apple',b='banbana'))
There apple and banbana
```

还可以通过获取字典直接传递数据, 像这样

```python
>>s=Template(There $a and $b)
>>d={'a':'apple','b':'banbana'}
>>print(s.substitute(d))
There apple and banbana
```

它们之间的差别在于对于参数缺少时的处理方式。

Template 的 实现方式是首先通过 Template 初始化一个字符串。这些字符串中包含了一个个 key。通过调用 substitute 或 safe_subsititute，将 key 值与方法中传递过来的参数对应上，从而实现在指定的位置导入字符串。这个方式的一个好处是不用像 print (‘%s’) 之类的方式，各个参数的顺序必须固定，只要 key 是正确的，值就能正确插入。通过这种方式，在插入很多数据的时候就可以松口气了。可是即使有这样偷懒的方法，依旧不能保证不出错，如果 key 少输入了一个怎么办呢？

substitute 是一个严肃的方法，如果有 key 没有输入，那就一定会报错。虽然会很难看，但是可以发现问题。

safe_substitute 则不会报错，而是将 $xxx 直接输入到结果字符串中，如

```
there apple and $b
```

这样的好处是程序总是对的，不用被一个个错误搞得焦头烂额。

Template 可以被继承, 它的子类可以进行一些‘个性化’操作…

通过修改 delimiter 字段可以将 $ 字符改变为其他字符，如 “#”，不过新的标示符需要符合正则表达式的规范。

通过修改 idpattern 可以修改 key 的命名规则，比如说规定第一个字符开头必须是 a，这对规范命名倒是很有好处。当然，这也是通过正则表示实现的。

```python
from string import Template


class MyTemplate(Template):
    delimiter = "#"
    idpattern = "[a][_a-z0-9]*"


def test():
    s='#aa is not #ab'
    t=MyTemplate(s)
    d={'aa':'apple','ab':'banbana'}
    print(t.substitute(d))


if __name__=='__main__':
    test()
```

