### 运行Flexx应用的方式

### 以桌面应用方式运行
开发期间创建一个 web 应用, 你将能使用`launch()`:
```
app = flx.App(MainComponent)
app.launch('app')  # to run as a desktop app
# app.launch('browser')  # to open in the browser
flx.run()  # mainloop will exit when the app is closed
```
### Jupyter中的Flexx
可以从 Jupyter notebook 交互式地使用 Flexx。使用`flx.init_notebook()`，它将注入必要的 JS 和 CSS。还可以使用`%gui asyncio`来启用 Flexx 事件系统。简单的部件(例如按钮)能很好地显示，但是对于其他部件，您可能需要使用`SomeWidget(minsize=300)`来指定尺寸大小。

到目前为止，Flexx还不支持JupyterLab.

### 作为一个web应用程序
可以提供 Flexx 应用程序，并允许多人连接。Flexx提供了让所有连接的客户端相互交互的方法，请参见聊天室和colab-paint示例。
```
app = flx.App(MainComponent)
app.serve('foo')  # Serve at http://domain.com/foo
app.serve('')  # Serve at http://domain.com/
flx.start()  # Keep serving "forever"
```
一些细节:

每个服务器进程驻留在一个URL(域 + 端口)上，但是可以服务多个应用程序(通过不同的路径)。每个进程使用一个 tornado IOLoop 和一个 tornado 应用程序对象。Flexx 的事件循环是基于asyncio (Tornado本身是集成asyncio)的。

当客户端连接到服务器时，将为其提供一个 HTML 页面，其中包含连接到websocket所需的信息。从这里开始，所有的通信都通过这个 websocket 进行，包括 CSS 和 JavaScript 模块的定义。

每个连接的开销都大于传统http框架的开销，Python-JS交互的复杂性可能会导致安全问题和内存泄漏。对于Flexx[演示页面](https://demo.flexx.app/)，我们在一个具有应用内存限制的自动重新启动Docker容器中运行服务器。

### 导出到静态web页面
任何应用程序都可以导出到其原始素材中。然而，依赖PyComponent的应用程序显然不能正常工作。对于使用会话/共享数据的应用程序来说，导出到单个文件是行不通的。
```
app = flx.App(MainComponent)
app.export('~/myapp/index.html')  # creates a few files
app.export('~/myapp.html', link=0)  # creates a single file
```

### 作为一个适当的web应用程序
当创建将长时间在服务器上运行 且/或 将被许多客户机访问的应用程序时，您可能希望使用http框架(如aiohttp、flask等)来实现服务器。

您可以首先将素材转储到字典中，使用您选择的库为其提供服务。有关完整的实现，请参见`serve_with_`示例。

值得注意的是，如果没有调用`App.serve()`、`flx.run()`和`flx.start()`，那么 Flexx 甚至不会导入 Tornado (我们有一个测试来确定它保持停留在此)。这使得在服务器启动时使用Flexx生成网站的客户端变得可行。
```
app = flx.App(MainComponent)
assets = app.dump('index.html')

...  # serve assets with flask/aiohttp/tornado/vibora/django/...
```