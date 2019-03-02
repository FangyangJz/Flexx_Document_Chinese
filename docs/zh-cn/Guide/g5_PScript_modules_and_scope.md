### PScript, 模块范围
在本指南中，我们已经看到了几个用JavaScript编写Python代码的例子。这是通过使用一个名为PScript的工具将Python代码转换成JavaScript来实现的，PScript是Flexx项目的一个派生工具。

### PScript几乎就是Python
PScript在语法上与Python兼容，因此可以在任何Python模块中编写它。PScript很像Python，将来可能会变得更好，但有时JavaScript会脱颖而出, 比如以下这些情况:
* 访问不存在的属性将返回`undefined`，而不是抛出AttributeError异常;
* 字典中的 key 被隐式转换为字符串;
* 类必须以大写字母开头，函数不能。这在Python中被很好的应用，但是PScript需以这种命名方式来区分类和函数;
* 如果函数具有`**kwargs`参数或在`*args`后命名的参数。将关键字传递给不处理关键字参
数的函数可能会导致混淆错误。
  
Python中不能做而这里可以做的事情:
* 以属性的方式访问字典中元素(例如, d.foo 代替 d['foo']);
* 通过将值添加到字符串中，隐式地将它们转换为 string 类型。
* 除以0 (导致 inf)

更多详细信息请参考 http://pscript.readthedocs.io/.

### 范围(Scope)
在Flexx中，很容易在同一个模块中定义 PyComponents 和 JsComponents 。为了清晰起见，对于较大的应用程序，最好避免这种情况。

在JsComponent的方法中，您可以使用同一个模块中定义的普通Python函数和类，前提是这些(及其依赖关系)可以由PScript进行转换。类似地，您可以使用在模块中定义或导入的对象。这些可以是整数、列表、dicts(以及它们的任何组合)，只要它可以是json序列化的。

对于JS中使用的每个Python模块，都会创建相应的JS模块。Flexx 会检测 JS 代码中使用了哪些变量名，没有在其中声明的，会试图在模块中找到对应的对象。您甚至可以从其他模块导入函数/类.
```
from flexx import flx

from foo import func1

def func2():
    ...

info = {'x': 1, 'y': 2}

class MyComponent(flx.JsComponent):

    @flx.reaction('some.event')
    def handler(self, *events):
        func1(info)
        func2()
```
在上面的代码中，Flexx 将在`MyComponent`的同一个模块中包含`func2`和`info`的定义，并包含JS模块`foo`中的`func1`。如果MyComponent不使用这些函数，JavaScript模块将不包含任何定义。

一个有用的特性是，可以在模块中使用PScript中的`RawJS`类来定义JS中的对象:
```
from flexx import flx

my_js_object = RawJS('window.something.get_some_object()')

class MyComponent(flx.JsComponent):

    @flx.reaction('some.event')
    def handler(self, *events):
        my_js_object.bar()
```
还可以将`__pscript__ = True`赋值给模块，以使Flexx将模块作为一个整体进行transpile。缺点是(目前为止)这样的模块不能使用导入。