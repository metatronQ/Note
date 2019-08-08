**运行时操作类：封装着JVM进程**  

* 取得实例：Runtime run = Runtime.getRuntime();
* 取得java虚拟机中的空闲内存量：run.freeMemory();
* 返回JVM的最大内存量：run.maxMemory();
* 运行垃圾回收器，释放空间:run.gc();
* 执行本机命令/程序:run.exec("a.exe");