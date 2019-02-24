### 部件(widgets)即组件(components)

部件就是我们所说的"组件", 是事件系统的核心. 事件系统允许部件有属性, 同时可以对应用中其他事物发生的变化做出响应(react), 我们将在下个章节讨论事件系统.

此时只要知道`Widget`类是一种`JsComponent`即可, 如你所想, 这意味着部件存在于 JavaScript 中. 但是非常酷的一件事是, 你可以在 Python 中使用部件, 可以设置部件的属性, 激活部件动作, 对部件的状态做出响应. 之所以可以这样, 是因为"代理对象"的存在.

### PyComponent 与 JsComponent

Flexx 应用是由存在于 Python 或者 JavaScript 中的组件构成, 在不同语言环境中的组件彼此可以通过一些方法进行通信.

`PyComponent` 和 `JSComponent` 类源自 `Component`类(下章一个专题介绍). 关于`PyComponent` 和 `JSComponent` 最重要的点如下:

* 他们都与会话`Session`有关系.
* 他们都有一个唯一的`id`属性在他们的会话`Session`中, 还有一个`uid`属性是全局唯一的.
* 他们分别存在于 Python 和 JavaScript 中(例如运行方法时)
* `PyComponent`只能Python中进行实例化, `JSComponent`既可以在Python中,也可以在JavaScript中进行实例化.
* `PyComponent`通常有一个对应的代理对象在 JavaScript 中
* `JSComponent`可能有代理对象在 Python 中, 如果当 Python 涉及到这些组件的时候, 会自动创建相应的代理对象.
  
实际使用中, 你将使用`PyComponent`去实现 Python 侧的行为, 用`JSComponent`(比如 Widgets部件)实现 JS 侧的行为. Flexx 允许一些连接 Python 和 JS 的方法, 但是这可能存在一个缺陷. 所以, 需要思考清楚你的 app 哪些部分是运行在Python中, 哪些运行在JS中, 非常重要. 入门指南后面我们将讨论用哪种模式可以帮助你.

### 代理组件

代理组件允许“另一方”检查属性、激活动作和连接到事件. 真正的组件知道代理对什么事件作出响应，并且只与这些事件通信.

下面的例子可能有点难以理解. 别担心, 在大多数情况下，事情应该是正常的.
```
from flexx import flx

class Person(flx.JsComponent):  # Lives in Js
    name = flx.StringProp(settable=True)
    age = flx.IntProp(settable=True)

    @flx.action
    def increase_age(self):
        self._mutate_age(self.age + 1)

class PersonDatabase(flx.PyComponent):  # Lives in Python
    persons = flx.ListProp()

    @flx.action
    def add_person(self, name, age):
        p = Person(name=name, age=age)
        self._mutate_persons([p], 'insert', 99999)

    @flx.action
    def new_year(self):
        for p in self.persons:
            p.increase_age()
```

在上面的代码中，`Person`对象位于 JavaScript 中，而数据库对象则位于 Python 中，它保存了`Person`对象的列表。实际上，`Person`组件在浏览器中会有一个可视化的表示。数据库也可以是`JsComponent`，但是我们假设在 Python 中需要它，因为它与 mysql 数据库或其他数据库同步。

我们可以观察到`add_person`动作 (在 Python 中执行) 实例化了新的`Person`对象。实际上，它实例化了代理对象，这些代理对象在 JavaScript 中自动获得相应的 (真实的) `Person`对象。`new_year`动作在 Python 中执行，而 Python 又调用每个人的`increase_age`动作，后者在 JavaScript 中执行。

JavaScript 也可以调用`PyComponents`的动作。对于上面的示例，我们必须将数据库对象放入`JsComponent`中。例如:
```
class Person(flx.JsComponent):
    ...
    def init(self, db):
        self.db = db
        # now we can call self.db.add_person() from JavaScript!

...

# To instantiate ...
Person(database, name=name, age=age)
```

### 根组件

另一个有用的特性是每个组件都有一个`root`属性，该属性包含对表示应用程序根的组件的引用。例如，如果`root`是一个`PersonDatabase`，那么所有`JsComponent`对象都有对这个数据库的引用(代理)。
