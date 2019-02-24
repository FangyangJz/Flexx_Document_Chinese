### 事件系统

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
<img src="https://github.com/FangyangJz/Flexx_Document_Chinese/blob/master/docs/_media/event.png?raw=true">
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
