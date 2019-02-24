# 命令行接口

Flexx 有一个命令行接口来执行一些简单的任务。通过`python -m flexx`调用它。可以提供额外的命令行参数来配置Flexx，请参见[配置Flexx](zh-cn/Reference/r6_Configuring_Flexx).

```
Flexx command line interface
  python -m flexx <command> [args]

help            show information on how to use this command.
info            show info on flexx server process corresponding to given port,
log             Start listening to log messages from a server process - STUB
stop            stop the flexx server process corresponding to the given port.
version         print the version number
```