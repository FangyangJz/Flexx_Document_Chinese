### 明智的可用模式
本章讨论一些可以用来构造应用程序的模式。哪种模式有意义取决于用例和个人偏好。此外，不要教条，这些只是为了给出一个合理的方向。

### 观察者模式
观察者模式的思想是观察者跟踪其他对象(的状态)，并且这些对象本身不知道它被跟踪的对象是什么。例如，在音乐播放器中，不需要编写代码来更新启动歌曲的函数中的窗口标题，而是有一个“当前歌曲”的概念，窗口将侦听当前歌曲的更改，以便在更改时更新标题。

这是一个几乎所有场景都应该遵循的模式。Flexx响应系统的设计使其尽可能的自然。这个想法也是下一种模式的核心.

### 使用中央数据存储
当应用程序的一个部分需要对应用程序的另一个部分中的某些内容作出反应时，可以创建直接连接到相关事件的响应。然而，随着应用程序的增长，这可能会让你感觉像意大利面条(盘根错节般复杂)。

最好定义一个表示应用程序状态的中心位置，例如在根组件上，或者在一个可以从根组件轻松访问的单独对象上。这样，应用程序的所有部分都是自包含的，可以在不需要更改应用程序其他地方的情况下进行自更新/替换。
```
from flexx import flx

class UserInput(flx.Widget):

    def init(self):
        with flx.VBox():
            self.edit = flx.LineEdit(placeholder_text='Your name')
            flx.Widget(flex=1)

    @flx.reaction('edit.user_done')
    def update_user(self, *events):
        self.root.store.set_username(self.edit.text)

class SomeInfoWidget(flx.Widget):

    def init(self):
        with flx.FormLayout():
            flx.Label(title='name:', text=lambda: self.root.store.username)
            flx.Widget(flex=1)

class Store(flx.JsComponent):

    username = flx.StringProp(settable=True)

class Example(flx.Widget):

    store = flx.ComponentProp()

    def init(self):

        # Create our store instance
        self._mutate_store(Store())

        # Imagine this being a large application with many sub-widgets,
        # and the UserInput and SomeInfoWidget being used somewhere inside it.
        with flx.HSplit():
            UserInput()
            flx.Widget(style='background:#eee;')
            SomeInfoWidget()
```

### 倾向于Python
如果您的应用程序是一个Python应用程序，恰好使用了 Flexx 而不是 Qt ，那么您可以通过使用 PyComponents 尽可能多地留在Python中。

我们重复上面的示例，但是现在大部分逻辑将在Python中实现。(结果几乎是一样的，但是如果我们在这个页面上显示它，它就不是reative了(不会工作)，因为没有Python。)
```
from flexx import flx

class UserInput(flx.PyComponent):

    def init(self):
        with flx.VBox():
            self.edit = flx.LineEdit(placeholder_text='Your name')
            flx.Widget(flex=1)

    @flx.reaction('edit.user_done')
    def update_user(self, *events):
        self.root.store.set_username(self.edit.text)

class SomeInfoWidget(flx.PyComponent):

    def init(self):
        with flx.FormLayout():
            self.label = flx.Label(title='name:')
            flx.Widget(flex=1)

    @flx.reaction
    def update_label(self):
        self.label.set_text(self.root.store.username)

class Store(flx.PyComponent):

    username = flx.StringProp(settable=True)

class Example(flx.PyComponent):

    store = flx.ComponentProp()

    def init(self):

        # Create our store instance
        self._mutate_store(Store())

        # Imagine this being a large application with many sub-widgets,
        # and the UserInput and SomeInfoWidget being used somewhere inside it.
        with flx.HSplit():
            UserInput()
            flx.Widget(style='background:#eee;')
            SomeInfoWidget()
```
### 只适合JS
如果您希望能够将应用程序作为静态页面发布到web上，则需要完全基于JsComponents。

对于服务于许多用户 和/或 长时间运行的web应用程序，也可以建议使用Flexx构建仅支持js的前端，并使用经典http框架(如aiohttp、flask等)实现后端。下一章将详细介绍如何做到这一点。

### 清晰分离
如果您的用例介于上述两种模式之间，那么您将更多地使用 PyComponents 和 JsComponents 的组合。

一般来说，最好清楚地将 Python 逻辑与 JavaScript 逻辑分开。例如，使用部件和 JsComponents 实现整个UI，使用一个或多个 PyComponents 实现“业务逻辑”。