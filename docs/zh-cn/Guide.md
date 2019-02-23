# 入门指南

## 基本部件 (Widgets)
如果你对Flexx感兴趣, 那么可能想做的第一件事情就是去创建一个 UI. 别急, 接下来就让我们看下它是如何工作的, 然后讨论下[组件 componets](), [事件 events]() 和 [反应 reactions]().

### 你的第一个部件
`Widget`类是其他 ui 类的基类, 本身作为抽象基类来使用.  通常是通过继承它来创建一个新的包含 ui 元素的部件.

```
from flexx import flx

class Example(flx.Widget):

    def init(self):
        flx.Button(text='hello')
        flx.Button(text='world')
```
[example01](https://flexx.readthedocs.io/en/stable/examples/exampleb3ff0faf9cb6687091a8ffb5cf76e7b6.html ':include :type=iframe width=100% height=100px')

上面这种写法的布局通常不是你想要的. 因此, 可以使用布局部件, 以一种更加明智的方式, 将空间分配给它包含的部分. 比如`HBox`:
```
from flexx import flx

class Example(flx.Widget):

    def init(self):
        with flx.HBox():
            flx.Button(text='hello', flex=1)
            flx.Button(text='world', flex=2)
```
<!-- [example02](https://flexx.readthedocs.io/en/stable/examples/example9686939190ff2292eb9d6089ad8c1085.html ':include :type=iframe width=100% height=100px') -->

`HBox`和`Button`也是部件. 我们上面创建的部件的例子涉及到了"复合部件(compound widgets)"; 部件里面包含部件. 这是创建新的 UI 元素最常用的方式.

### 构建部件

复合部件可以被用在你的app中任何地方. 它们通过执行`init()`方法被创建. 在这个方法中, 部件默认作为父类.

任何`widget`类也可以作为上下文管理器 (*context manager*) 来使用.

> ContextManager, 上下文是 context 直译的叫法,在程序中用来表示代码执行过程中所处的前后环境。上下文管理器中有 __enter__ 和 __exit__ 两个方法，以with为例子，__enter__ 方法会在执行 with 后面的语句时执行，一般用来处理操作前的内容。比如一些创建对象，初始化等；__exit__ 方法会在 with 内的代码执行完毕后执行，一般用来处理一些善后收尾工作，比如文件的关闭，数据库的关闭等。

在上下文中的部件默认为父类, 任何在上下文中创建的部件, 如果没有指定父类, 都视其为父类. (这种机制是线程安全的) 这种方式使你的 app 结构更加清晰.
```
from flexx import flx

class Example(flx.Widget):

    def init(self):
        with flx.HSplit():
            flx.Button(text='foo')
            with flx.VBox():
                flx.Widget(style='background:red;', flex=1)
                flx.Widget(style='background:blue;', flex=1)
```

### 部件变成 app

把一个部件 widget 变成 app 只要简单的将部件打包进`App`即可. 之后调用`launch()`启动桌面 app, `serve()`启动 web app, `dump()`导出 assets, `export()` 可以导出一个单独的 HTML 文件, 甚至可以`publish()`直接在线发布 ( 该功能处于试验阶段 ). 在入门指南的后面部分, 我们将进一步讲解不同的启动 app 方式.

```
from flexx import flx

class Example(flx.Widget):
    def init(self):
        flx.Label(text='hello world')

app = flx.App(Example)
app.export('example.html', link=0)  # Export to single file
```
如果想展示 app, 使用 launch:
```
app.launch('browser')  # show it now in a browser
flx.run()  # enter the mainloop
```

### Python 方法使用部件

上面的例子中, 我们已经用"经典"方法从基础组件去创建应用. Flexx 提供了很多 布局(layout)部件 和 叶子(leaf)部件, 比如 控件(controls), 详细请查阅 [部件类列表](https://flexx.readthedocs.io/en/stable/ui/api.html)

### Web 方法使用部件

源自 React 使用 html 元素创建自定义部件的灵感, 对于 web 开发者来说, 可能更加熟悉这种方法. 如果你是使用 Python 的, 下面这个例子对于你来说看上去很奇怪, 但是别担心, 你可以不使用这种方式:
```
from flexx import flx

class Example(flx.Widget):

    name = flx.StringProp('John Doe', settable=True)
    age =  flx.IntProp(22, settable=True)

    @flx.action
    def increase_age(self):
        self._mutate_age(self.age + 1)

    def _create_dom(self):
        # Use this method to create a root element for this widget.
        # If you just want a <div> you don't have to implement this.
        return flx.create_element('div')  # the default is <div>

    def _render_dom(self):
        # Use this to determine the content. This method may return a
        # string, a list of virtual nodes, or a single virtual node
        # (which must match the type produced in _create_dom()).
        return [flx.create_element('span', {},
                    'Hello', flx.create_element('b', {}, self.name), '! '),
                flx.create_element('span', {},
                    'I happen to know that your age is %i.' % self.age),
                flx.create_element('br'),
                flx.create_element('button', {'onclick': self.increase_age},
                    'Next year ...')
                ]
```
`_render_dom()`方法被一个 reaction 隐式调用. 这就意味着, 当在此函数期间访问的任何属性发生变化时, 这个函数会自动再被调用一次. 因此, 这提供了一个非公开的方式, 使用 HTML 元素去定义一个部件特性.

以上的例子中, 在`create_element()`中的第三个参数是一个字符串, 其实也可以是一个字典元素的列表( `create_element()`返回一个字典)


## 部件(widgets)即组件(components)

部件就是我们所说的"组件", 是事件系统的核心. 事件系统允许部件有属性, 同时可以对应用中其他事物发生的变化做出响应(react), 我们将在下个章节讨论事件系统.

此时只要知道`Widget`类是一种`JsComponent`即可, 如你所想, 这意味着部件存在于 Javascript 中. 但是非常酷的一件事是, 你可以在 Python 中使用部件, 可以设置部件的属性, 激活部件动作, 对部件的状态做出响应. 之所以可以这样, 是因为"代理对象"的存在.

### PyComponent 与 JsComponent

Flexx 应用是由存在于 Python 或者 Javascript 中的组件构成, 在不同语言环境中的组件彼此可以通过一些方法进行通信.

`PyComponent` 和 `JSComponent` 类源自 `Component`类(下章一个专题介绍). 关于`PyComponent` 和 `JSComponent` 最重要的点如下:

* 他们都与会话`Session`有关系.
* 他们都有一个唯一的`id`属性在他们的会话`Session`中, 还有一个`uid`属性是全局唯一的.
* 他们分别存在于 Python 和 Javascript 中(例如运行方法时)
* `PyComponent`只能Python中进行实例化, `JSComponent`既可以在Python中,也可以在Javascript中进行实例化.
* `PyComponent`通常有一个对应的代理对象在 Javascript 中
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


## 事件系统

### 与 Flexx 其他部分的关系

### 事件对象

### 组件类

### 