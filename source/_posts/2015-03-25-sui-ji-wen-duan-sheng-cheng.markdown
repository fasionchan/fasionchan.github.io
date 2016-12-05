---
layout: post
title: "随机文段生成"
date: 2015-03-25 20:40:54 +0800
comments: true
categories: 
---

我们经常需要拷贝一些文字作为Demo填充，特别是做页面时，每次都到网上去找觉得挺麻烦的。很多时候我们也需要一个随机的信息流，比如模拟后台日志输出。Python下[loremipsum](https://pypi.python.org/pypi/loremipsum/)就可以很好地完成这个工作，安装很简单：`pip install loremipsum`。使用示例如下：

```
>>> import loremipsum
>>> dir(loremipsum)
['DictionaryError', 'Generator', 'SampleError', '_GENERATOR', '__all__', '__author__', '__builtins__', '__classifiers__', '__copyright__', '__doc__', '__docformat__', '__file__', '__keywords__', '__name__', '__package__', '__path__', '__version__', 'generate_paragraph', 'generate_paragraphs', 'generate_sentence', 'generate_sentences', 'generator', 'get_paragraph', 'get_paragraphs', 'get_sentence', 'get_sentences']
>>>
>>> # 生成一个随机句子
>>> print loremipsum.get_sentence()
Purus neque fames vivamus.
>>>
>>> # 生成一个随机段落
>>> print loremipsum.get_paragraph(10)
Lorem ipsum. Vitae massa aenean mi pulvinar montes tortor felis feugiat. Morbi risus. Etiam class tortor parturient volutpat dolor a integer mollis. Neque felis. Fusce ipsum massa dui purus penatibus phasellus fusce phasellus cursus. Felis augue nunc egestas natoque dictum dolor mus. Massa metus donec eni nibh habitasse ut tortor mi orci aliquet risus in. Lorem augue per tellus convallis facilisi eu purus suspendisse. Velit purus sem felis tincidunt cras. Fusce neque luctus magnis fusce. Class dolor egestas hac eleifend tincidunt. Lacus neque ligula sollicitudin facilisis tortor inceptos non id nunc.
```
