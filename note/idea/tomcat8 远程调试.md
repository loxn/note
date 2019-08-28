https://www.jianshu.com/p/f9b843f16744

1. 修改tomcat的bin目录下的catalina.sh加入下面的代码

   ```
   JPDA_OPTS="-Xrunjdwp:transport=dt_socket,address=8000,server=y,suspend=n"
   注意: address为debug的端口号,要在系统中放开这个端口号(云服务器要在相关安全规则中配置)
   ```

   

2. 使用命令启动tomcat

   ```
   bin/catalina.sh jpda start
   ```

3. 配置idea

![1561341539944](D:\笔记\img\1561341539944.png)