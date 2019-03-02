### 响应 (Reactions)
`Reactions`被用于响应事件和属性变化, 它使用的是一个底层的处理函数:
```
from flexx import flx

class Example(flx.Widget):

    def init(self):
        super().init()
        with flx.VBox():
            with flx.HBox():
                self.firstname = flx.LineEdit(placeholder_text='First name')
                self.lastname = flx.LineEdit(placeholder_text='Last name')
            with flx.HBox():
                self.but = flx.Button(text='Reset')
                self.label = flx.Label(flex=1)

    @flx.reaction('firstname.text', 'lastname.text')
    def greet(self, *events):
        self.label.set_text('hi ' + self.firstname.text + ' ' + self.lastname.text)

    @flx.reaction('but.pointer_click')
    def reset(self, *events):
        self.label.set_text('')
```
这个例子演示了多个概念。首先，这些响应通过指定事件类型的连接字符串进行连接; 在本例中，`greet()`响应与“firstname.text”和“lastname.text”相关联，`reset()`连接到按钮的事件类型为“pointer_click”的事件。可以看到连接字符串是一个路径，例如“sub.subsub.event_type”。这种方式允许使用一些强大的机制，在动态性一节中会进行讨论。

可以看到响应函数接收`*event`参数, 这是因为响应能传递 0个 或 更多个事件. 如果一个响应被人为调用 (例如 `ob.handle_foo()`), 它含有0个事件. 当被事件系统调用, 响应至少含有1个事件. 例如，当一个属性被设置两次时，这个函数只会被调用一次，但是会有多个事件。如果所有事件都需要单独处理，请使用:
```
@flx.reaction('foo')
def handler(self, *events):
    for ev in events:
        ...
```
在大多数情况下，您将连接到预先知道的事件，比如与属性和发射器对应的事件。如果您连接到一个未知的事件，Flexx将显示一个警告, 可以使用`!foo`来阻止这些警告。

### 贪婪和自动响应
每个响应都以一定的“模式”进行。在模式“normal”(默认)中，事件系统确保所有事件按照发出的顺序处理。这通常是最明智的方法，但这意味着在单个事件在循环迭代期间可以多次调用同一个响应，在这期间其他响应也被调用，以确保事件顺序的一致性。

如果希望以一个响应为目标的所有事件, 都通过对该响应的单个调用来处理，那么可以将其设置为“贪婪(greedy)”模式。当所有相关事件都必须同时处理时，或者是性能非常重要而顺序不那么重要时，这种模式才有意义。
```
@flx.reaction('foo', mode='greedy')
def handler(self, *events):
    ...
```
具有“auto”模式的响应, 在其使用的任何属性发生变化时, 响应会自动触发。可以通过指定`mode`参数来创建此类响应，或者简单地创建一个没有连接字符串的反应。我们把这种反应称为“自动反应(auto reactions)”或“内隐反应(implicit reactions)”。

这是一个非常方便的特性，但是它的开销比正常的响应要大，因此在访问大量属性或使用的属性经常更改时，应该避免使用它。很难确切地说它什么时候开始显著地影响性能，但是“通常”可能在数百次左右，“通常在每秒100次左右”。请记住这一点，并在需要时进行基准测试。

```
from flexx import flx

class Example(flx.Widget):

    def init(self):
        super().init()
        with flx.VBox():
            with flx.HBox():
                self.slider1 = flx.Slider(flex=1)
                self.slider2 = flx.Slider(flex=1)
            self.label = flx.Label(flex=1)

    @flx.reaction
    def sliders_combined(self):
        self.label.set_text('{:.2f}'.format(self.slider1.value + self.slider2.value))
```
一个类似的有用特性是(在初始化时)使用函数分配属性。在这种情况下，函数变成了隐式反应。这可以方便地连接应用程序的不同部分。
```
from flexx import flx

class Example(flx.Widget):

    def init(self):
        super().init()
        with flx.VBox():
            with flx.HBox():
                self.slider1 = flx.Slider(flex=1)
                self.slider2 = flx.Slider(flex=1)
            self.label = flx.Label(flex=1, text=lambda:'{:.2f}'.format(self.slider1.value * self.slider2.value))
```
### 表组内突变的响应
列表或数组的内部突变可以通过逐个处理事件来响应
```
class MyComponent(flx.Component):

    @flx.reaction('other.items')
    def track_array(self, *events):
        for ev in events:
            if ev.mutation == 'set':
                self.items[:] = ev.objects
            elif ev.mutation == 'insert':
                self.items[ev.index:ev.index] = ev.objects
            elif ev.mutation == 'remove':
                self.items[ev.index:ev.index+ev.objects] = []  # objects is int here
            elif ev.mutation == 'replace':
                self.items[ev.index:ev.index+len(ev.objects)] = ev.objects
            else:
                assert False, 'we cover all mutations'
```
为了方便起见，还可以使用`flx.mutate_array()`和`flx.mutate_dict()`函数“复制”该突变。

### 连接字符串标签
连接字符串可以使用标签来影响调用响应的顺序，并且可以提供立即断开特定(组)处理程序的方法
```
class MyObject(flx.Component):

    @flx.reaction('foo')
    def given_foo_handler(*events):
            ...

    @flx.reaction('foo:aa')
    def my_foo_handler(*events):
        # This one is called first: 'aa' > 'given_f...'
        # 前一个响应没有设置key, 也就是标签, 那么key就是函数名given_foo_handler
        # 这里的响应设置了key为aa, aa比given_foo_handler排序靠前, 所以先被调用
        ...
```
当发出一个事件时，任何连接的响应都按照键(如果有的话就是标签)的顺序被调度，否则就是反应的名称.

标签也可以用在`disconnect()`方法中:
```
@h.reaction('foo:mylabel')
def handle_foo(*events):
    ...

...

h.disconnect('foo:mylabel')  # don't need reference to handle_foo
```

### 动态机制(Dynamism)
动态机制的概念是指 允许连接到 源可以变化的事件。在下面的示例中，我们连接到按钮列表的click事件，即使列表发生变化，该事件也会继续工作。
```
from flexx import flx

class Example(flx.Widget):

    def init(self):
        super().init()
        with flx.VBox():
            with flx.HBox():
                self.but = flx.Button(text='add')
                self.label = flx.Label(flex=1)
            with flx.HBox() as self.box:
                flx.Button(text='x')

    @flx.reaction('but.pointer_click')
    def add_widget(self, *events):
        flx.Button(parent=self.box, text='x')

    @flx.reaction('box.children*.pointer_click')
    def a_button_was_pressed(self, *events):
        ev = events[-1]  # only care about last event
        self.label.set_text(ev.source.id + ' was pressed')
```
`a_button_was_pressed`在单击框中的任何按钮时调用。当盒子的子元素发生变化时，响应会自动重新连接。注意，在某些情况下，您可能还希望连接到`box.children`的属性的更改。

以上有效果是因为`box.children`是一个属性, 如果它连接到的是常规列表中的部件，那么这个响应仍然有效，但是它不是动态的。

### 隐式动态机制(Implicit dynamism)
内隐响应也是动态的，甚至可能更动态! 在下面的示例中，该响应访问子属性，因此每当该属性发生变化时，它将被调用。它还连接到所有子元素的可见事件，以及所有可见子元素的`foo`事件。
```
from flexx import flx

class Example(flx.Widget):

    def init(self):
        super().init()
        with flx.VBox():
            with flx.HBox():
                self.but = flx.Button(text='add')
                self.label = flx.Label(flex=1)
            with flx.HBox() as self.box:
                flx.CheckBox()

    @flx.reaction('but.pointer_click')
    def add_widget(self, *events):
        flx.CheckBox(parent=self.box)

    @flx.reaction
    def a_button_was_pressed(self):
        ids = []
        for checkbox in self.box.children:
            if checkbox.checked:
                ids.append(checkbox.id)
        self.label.set_text('checked: ' + ', '.join(ids))
```
这种机制非常强大，但是我们可以看到它是如何潜在地访问(从而连接)许多属性的，特别是当响应调用其他访问更多属性的函数时。如前所述，请记住，隐式反应的开销更大，它随所访问属性的数量而增加。