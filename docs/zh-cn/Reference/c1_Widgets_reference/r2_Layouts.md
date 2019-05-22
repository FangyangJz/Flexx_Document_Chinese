# Widget

提供基础`Widget`类.

当子类化部件来创建复合部件 (即包含其他部件的部件) 时, 在`init()`方法中初始化子部件。当部件是当前部件时调用该方法; 在其中实例化的任何部件都将自动成为子部件.
```
from flexx import flx

class Example(flx.Widget):
    def init(self):
        super().init()

        flx.Button(text='foo')
        flx.Button(text='bar')
```



# FormLayout
在表单中布局一系列(输入)部件。例子:
```
from flexx import ui

class Example(ui.Widget):
    def init(self):
        with ui.FormLayout():
            self.b1 = ui.LineEdit(title='Name:')
            self.b2 = ui.LineEdit(title="Age:")
            self.b3 = ui.LineEdit(title="Favorite color:")
            ui.Widget(flex=1)  # Spacing
```
`class flexx.ui.layouts._form.BaseTableLayout(*init_args, **kwargs)`
继承自`Layout`
使用HTML表格布局的抽象基类.

使用这种方法的布局在调整大小时性能不好。当它被用作叶子布局时，这并不是一个问题，但是不建议将这样的布局嵌入到其他布局中。

> JS 部分别名 `BaseTableLayout`

`class flexx.ui.FormLayout(*init_args, **kwargs)`
继承自`BaseTableLayout`

一个布局部件，它在窗体中垂直地分解子部件。在每个部件的左侧放置一个标签(基于部件的标题)。

这个部件的节点是。(可以改为使用CSS布局)

> JS 部分别名 `FormLayout`

# HVLayout
HVLayout及其子类为水平或垂直堆栈部件提供了一种简单的机制。这可以在不同的模式下完成:box模式适用于调整内容的自然大小。fix模式和split模式更适合高级布局。有关详细信息，请参见HVLayout类。

交互框布局示例:
```
from flexx import app, event, ui

class Example(ui.HBox):
    def init(self):
        self.b1 = ui.Button(text='Horizontal', flex=0)
        self.b2 = ui.Button(text='Vertical', flex=1)
        self.b3 = ui.Button(text='Horizontal reversed', flex=2)
        self.b4 = ui.Button(text='Vertical reversed', flex=3)

    @event.reaction('b1.pointer_down')
    def _to_horizontal(self, *events):
        self.set_orientation('h')

    @event.reaction('b2.pointer_down')
    def _to_vertical(self, *events):
        self.set_orientation('v')

    @event.reaction('b3.pointer_down')
    def _to_horizontal_rev(self, *events):
        self.set_orientation('hr')

    @event.reaction('b4.pointer_down')
    def _to_vertical_r(self, *events):
        self.set_orientation('vr')
```

`class flexx.ui.HVLayout(*init_args, **kwargs)`

### HBox
`class flexx.ui.HBox(*init_args, **kwargs)`

继承自`HVLayout`

水平布局，尝试给每个部件分配其自然大小，并分配与部件的flex值相对应的任何剩余空间。(即 HVLayout，方向为“h”，模式为“box”)

### HFix
`class flexx.ui.HFix(*init_args, **kwargs)`

继承自`HVLayout`

### HSplit
`class flexx.ui.HSplit(*init_args, **kwargs)`

继承自`HVLayout`

### VBox
`class flexx.ui.VBox(*init_args, **kwargs)`

继承自`HVLayout`

### VFix
`class flexx.ui.VFix(*init_args, **kwargs)`

继承自`HVLayout`

### VSplit
`class flexx.ui.VSplit(*init_args, **kwargs)`

继承自`HVLayout`
