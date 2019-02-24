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
