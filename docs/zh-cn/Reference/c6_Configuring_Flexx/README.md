# 配置 Flexx

这个页面列出了通过`Config`类实现的 Flexx 配置选项。配置选项从`<appdata>/.flexx.cfg` (检查`flexx.util.config.appdata_dir()`中的实际位置) 读取，也可以使用环境变量和命令行参数进行设置，如下所示。或者，可以通过`flexx.config.foo=3`在 Python 中直接设置选项。

### flexx.config

flexx的配置对象

选项(options) 可以从不同的来源进行设置，并按以下顺序计算:

* 默认值
* .cfg 或者 .ini 文件, 或者 cfg 格式的字符串
* 环境变量, 例如 `FLEXX_FOO=3`
* 命令行参数, 例如 `--flexx-foo=3`
* 直接赋值配置选项, 例如 `config.foo = 3`

可以使用`print(config)`获取当前值的摘要以及它们是从哪个源设置的。

<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;}
.tg td{font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;border-color:black;}
.tg th{font-family:Arial, sans-serif;font-size:14px;font-weight:normal;padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;border-color:black;}
.tg .tg-0lax{text-align:left;vertical-align:top}
.tg .tg-0pky{border-color:inherit;text-align:left;vertical-align:top}
</style>
<table class="tg">
  <tr>
    <th class="tg-0lax">参数列表</th>
    <th class="tg-0lax">值类型</th>
    <th class="tg-0lax">作用描述</th>
  </tr>
  <tr>
    <td class="tg-0lax">browser_stacktrace</td>
    <td class="tg-0lax">bool</td>
    <td class="tg-0pky">在浏览器窗口中显示服务器堆栈跟踪(默认为True)<br></td>
  </tr>
  <tr>
    <td class="tg-0lax">cookie_secret</td>
    <td class="tg-0lax">str</td>
    <td class="tg-0pky">编码cookie的密钥。(默认“flexx_secret”)<br></td>
  </tr>
  <tr>
    <td class="tg-0lax">host_whitelist</td>
    <td class="tg-0lax">str</td>
    <td class="tg-0pky">允许 &lt;host&gt;:&lt;port&gt; 通过跨域检查, 域之间用逗号分隔。(默认'')<br></td>
  </tr>
  <tr>
    <td class="tg-0lax">hostname</td>
    <td class="tg-0lax">str</td>
    <td class="tg-0pky">服务应用程序的默认主机名。(默认“localhost”)<br></td>
  </tr>
  <tr>
    <td class="tg-0lax">log_level</td>
    <td class="tg-0lax">str</td>
    <td class="tg-0pky">要使用的日志级别(DEBUG, INFO, WARNING, ERROR)(默认 “info”)<br></td>
  </tr>
  <tr>
    <td class="tg-0lax">port</td>
    <td class="tg-0lax">int</td>
    <td class="tg-0lax">应用服务的默认端口, 0表示自动选择. (默认 0)</td>
  </tr>
  <tr>
    <td class="tg-0lax">ssl_certfile</td>
    <td class="tg-0lax">str</td>
    <td class="tg-0lax">https服务器的证书文件. (默认 '')</td>
  </tr>
  <tr>
    <td class="tg-0lax">ssl_keyfile</td>
    <td class="tg-0lax">str</td>
    <td class="tg-0lax">https服务器的密钥文件。(默认 '')</td>
  </tr>
  <tr>
    <td class="tg-0lax">tornado_debug</td>
    <td class="tg-0lax">bool</td>
    <td class="tg-0lax">设置tornado应用程序调试标志, 允许自动加载和其他调试特性。(默认“false”)</td>
  </tr>
  <tr>
    <td class="tg-0lax">webruntime</td>
    <td class="tg-0lax">str</td>
    <td class="tg-0lax">使用默认web runtime。默认是“app或浏览器”。(默认 '')</td>
  </tr>
  <tr>
    <td class="tg-0lax">ws_timeout</td>
    <td class="tg-0lax">int</td>
    <td class="tg-0lax">如果websocket在这段时间内处于空闲状态，那么它将被关闭。(默认 20)</td>
  </tr>
</table>