# 入门指南

## 基本部件 (Widgets)
如果你对Flexx感兴趣, 那么可能想做的第一件事情就是去创建一个 UI. 别急, 接下来就让我们看下它是如何工作的, 然后讨论下[组件 componets](), [事件 events]() 和 [响应 reactions]().

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


## 事件系统

事件系统由 组件(components), 属性(properties), 事件(events) 和 响应(reactions) 组成. 它们使得一个应用的不同组件可以彼此相互作用(react), 也可以根据用户输入做出响应(react).

简单来说:
* 组件`components`(或者 widgets)是构成应用的基本单元.
* 每个组件都拥有表示组件状态的属性`properties`.
* 属性只能通过动作(`action`)进行突变。调用 (即激活) 某个动作不会立即执行该动作; 动作是分批处理的.
* 当属性被修改 (即状态改变) 时，相应的响应 (reactions) 将被调用。当所有挂起的动作 (actions) 都完成时，将处理这些响应 (reactions). 这意味着在处理响应 (reactions) 的过程中，状态永远不会改变，这是一个非常值得依赖的东西!
* 响应 (reactions) 还可以对发射器`emitters`生成的事件(如鼠标事件)作出响应.
* 事件循环`event loop`对象负责调度动作 (actions) 和 响应 (reactions). 在 Python 中, 它与 Python 自己的异步循环 (asyncio loop) 集成. 在JavaScript中，它利用了JavaScript的调度机制.

动作的异步性, 加上在处理响应过程中状态不会发生变化, 使得我们很容易对因果关系进行推理. 信息是单向流动的, 这个概念借鉴了现代框架, 如 React/Flux 和 Veux.

<div align=center>
<img src="/../_media/event.png">
</div>

有人可能会说, 信息流仍然是循环的, 因为有一个箭头从响应 (reactions) 指向动作 (actions), 这是正确的, 但是请注意, 从响应激活的动作不是直接执行的; 它们会被挂起, 只有在所有响应完成后才会被执行.

### 与 Flexx 其他部分的关系

事件系统及其组件`Component`类构成了`flexx.ui`中的`app.PyComponent`, `app.JsComponent`和 UI系统 的基础. 它可以在 Python 和 JavaScript 中使用，并且在两种语言中工作完全相同.

除此之外, 这是一个通用的事件系统, 可以驱动任何基于asyncio的系统.

### 事件对象

事件是在特定时间点发生的事情, 例如鼠标被按下或属性改变其值. 在Flexx中, 事件用字典(dictionary)对象表示, 字典对象提供关于事件的信息 (例如按了什么按钮, 或者属性的新旧值). 使用自定义`Dict`类需要继承自`dict`, 允许属性访问, 例如`ev.button`, 按钮作为`ev['button']`的替代品.

每个事件对象至少有两个属性 : `source`源(发出事件的组件对象的引用) 和 `type`类型(指示事件类型的字符串)

### 组件类

`Component`类为具有属性、动作、响应 和 发射器 的对象提供基类。你可以像这样创建你自己的组件:
```
class MyObject(flx.Component):
    ...  # attributes/properties/actions/reactions/emitters 在这里创建

    def init(self):
        super().init()
        ...
```
实现组件类的`init()`方法是通用的. 当所有属性都已初始化, 但还没有发出事件时, 组件会自动调用`init()`, 此时是进一步初始化组件 [和/或] 实例化子组件的好时机. 很少需要实现`__init__()`方法.

调用`init()`时, 组件是当前的 "活动(active)" 组件, 可以使用它来描述对象的层次结构，就像使用 部件(widgets) 一样. 这还意味着允许进行更改, 并且组件本身上的 动作(actions) 可以直接生效 (尽管激活其他组件的动作仍然是异步的).

让我们看一个实际工作的 部件(widget) 示例并将其分解. 它包含一个属性、一个动作和几个响应:
```
from flexx import flx

class Example(flx.Widget):

    counter = flx.IntProp(3, settable=True)

    def init(self):
        super().init()

        with flx.HBox():
            self.but1 = flx.Button(text='reset')
            self.but2 = flx.Button(text='increase')
            self.label = flx.Label(text='', flex=1)  # take all remaining space

    @flx.action
    def increase(self):
        self._mutate_counter(self.counter + 1)

    @flx.reaction('but1.pointer_click')
    def but1_clicked(self, *events):
        self.set_counter(0)

    @flx.reaction('but2.pointer_click')
    def but2_clicked(self, *events):
        self.increase()  # reaction响应 激活 increase()动作 激活 _mutate_counter(+1)动作, 
                        # 响应激活动作是异步的, 动作激活动作马上执行

    @flx.reaction
    def update_label(self, *events):
        self.label.set_text('count is ' + str(self.counter))
```
我们将进一步介绍属性和动作. 响应是如此的酷, 以至于他们有了[自己的篇章](https://flexx.readthedocs.io/en/stable/guide/reactions.html) : )

### 属性表现状态

在上面部件的例子中, 我们看到了一个`int`属性. 还有很多不同的属性类型`property types`.例如:
```
class MyObject(flx.Component):

    foo = flx.AnyProp(8, settable=True, doc='can have any value')
    bar = flx.IntProp()
```
属性使用一个参数来设置默认值. 如果没有给出, 则使用一个合理的默认值, 该值取决于属性的类型. 可以使用`doc`参数来添加文档说明. 注意, 属性是只读的 : 它们只能通过 动作(actions) 进行变更. `foo`属性(以及`counter`属性)被标记为`settable`，这将自动创建`set_foo()`动作.  

当一个组件被创建的时候, 属性值(即使设置为`non-settable`)可以被初始化:
```
c = MyComponent(foo=42)
```
还可以将属性的初值设置为函数对象. 这将创建一个设置属性的自动响应(auto-reaction), 并使得简洁连接事物成为可能, 在下面的示例中, 当用户名属性发生变化时, 标签文本将自动更新:
```
flx.Label(flex=1, text=lambda: 'count is ' + str(self.counter))
```
每当属性发生更改时, 都会发出一个事件. 此事件具有属性`old_value`和`new_value`(除了位置数组[in-place array]改变, 接下来会提到). 

在初始化时, 组件为每个属性发送一个事件, 其中`old_value`和`new_value`是相同的.

### Attributes 属性

`Component`类也有`Attributes`属性, 这些属性是只读的 (通常是静态的), 值是非可见的 (比如 `JsComponent.id`)

### 本地属性

`JsComponent`的常规方法只在 JavaScript 中可用. 另一方面, 代理对象上的所有属性也是可用的, 但实际情况可能并不总是这样. 

可以使用`LocalProperty`创建 JavaScript (或`PyComponent`中的 Python )本地的属性. 

另一种选择可能是使用`Attributes`属性; 它们也是 JavaScript/Python中 本地的属性

### 动作(actions) 改变 属性(properties)

在上面部件的例子中, 我们看到了`increase()`动作的定义. `Actions`是必须的, 因为它们是唯一可以对 属性(properties) 进行改变的地方.
```
class Example(flx.Widget):

    counter = flx.IntProp(3, settable=True)

    ...

    @flx.action
    def increase(self):
        self._mutate_counter(self.counter + 1)
```
你可能好奇为什么例子中的 响应(reaction) 没有直接`self.set_counter(self.counter + 1)`. 这是因为 动作(actions) 是异步的; 激活一个动作并不会马上表现出来, 因此激活`set_counter()`这个动作两次将只能适用于最后一次的值. 但是请注意, 当从另一个动作调用某个动作时, 它是直接执行的.

动作(action) 可以有任意数量的 (位置) 参数, 并且总是返回组件本身, 这就允许action 链式调用, 例如` t.scale(3).translate(3, 4)`.

通过`_mutate`方法或自动生成的`_mutate_xx()`方法可以进行属性更新. 更新只能通过一个动作来完成, 否则将导致错误. 乍一看, 这似乎有些限制, 但它极大地简化了对应用程序中信息流动的推理, 即使在应用程序扩展时也是如此.

### 数组属性的更新

上面例子都是简单而常用的属性更新. 对于列表属性`list properties`, 更新可以就地进行:
```
from flexx import flx

class Example(flx.Widget):

    items = flx.ListProp(settable=True)

    def init(self):
        super().init()

        with flx.HBox():
            self.but1 = flx.Button(text='reset')
            self.but2 = flx.Button(text='add')
            flx.Label(flex=1, wrap=2, text=lambda: repr(self.items))

    @flx.action
    def add_item(self, item):
        self._mutate_items([item], 'insert', len(self.items))  # Here

    @flx.reaction('but1.pointer_click')
    def but1_clicked(self, *events):
        self.set_items([])

    @flx.reaction('but2.pointer_click')
    def but2_clicked(self, *events):
        self.add_item(int(time()))
```
这允许对状态更新进行更细粒度的控制, 而状态更新也可以通过更有效的方式由响应来处理. 可更新的类型有`“set”(默认)`, `“insert”`, `“replace”`和`“remove”`. 在`remove`中，提供的值是要删除的元素的数量. 对于其他元素, 提供的信息必须是在指定索引处`set/insert/replace`的元素列表.

### 发射器(emitter) 创建事件

`Emitters` 使生成事件变得容易. 与动作类似, 它们也是用装饰器创建的.
```
# Somewhere in the Flexx codebase:
class Widget(JsComponent):

    ...

    @flx.emitter
    def key_down(self, e):
        """ Event emitted when a key is pressed down while this
        widget has focus.
        ...
        """
        return self._create_key_event(e)
```
发射器可以有任意数量的参数, 并且应该返回一个字典, 该字典将作为事件发出, 其事件类型与发射器的名称匹配.

请注意, 严格地说, 发射器不是必需的, `Component.emit()`可用于生成事件. 但是, 它们提供了一种机制来根据特定的输入数据生成事件, 并记录组件可能发出的事件.

## 响应 (Reactions)
