### Widgets
如果你对Flexx感兴趣, 那么可能想做的第一件事情就是去创建一个 UI. 别急, 接下来就让我们看下它是如何工作的, 然后讨论下[组件 componets](/zh-cn/Guide/g2_Widgets_are_components), [事件 events](/zh-cn/Guide/g3_The_event_system) 和 [响应 reactions](/zh-cn/Guide/g4_Reactions).

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
