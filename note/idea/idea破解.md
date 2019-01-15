https://blog.csdn.net/zty1317313805/article/details/81503550

找到`idea.exe.vmoptions`和`idea64.exe.vmoptions`，用notepad++（或记事本）打开它们。

最后一行加上下面的：

-javaagent:-javaagent:..\JetbrainsCrack-2.9-release-enc.jar