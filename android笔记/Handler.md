# 处理消息的手段——Handler、Looper、MessageQueue

* **Handler：创建在主线程，用于更新UI，每个Handler都要关联一个消息队列**


* **Looper：创建消息死循环，并将消息队列与线程关联上，这样不同线程就不能访问对方的消息队列**


* **MessageQueue：封装在Looper对象中，用于存取消息的队列**

## 过程：

**在ActivityTread.main方法中通过Looper.prepareMainLooper()创建消息循环Looper，并将消息队列与线程绑定，之后调用Looper.loop()执行消息循环；在主线程中创建Handler实例，调用发送的方法将消息发送给消息队列，消息队列又将消息分发给Handler处理，此时Handler位于主线程，可以进行更新UI的操作**

## Handler一般不能创建在子线程中（更新UI一定在主线程）

**如果在子线程中创建Handler实例会抛出异常，因为Handler创建时需要绑定MessageQueue，而MessageQueue被封装在Looper中，而在子线程创建时，Looper对象是没有被创建的（主线程中的Looper是随主函数被创建的），Looper为null，此时在子线程中Handler就绑定不到Looper，就会抛出异常；若要在子线程中使用Handler，则需要手动调用Loop.prepare()方法对Looper进行创建，并调用Looper.loop()方法启动消息循环（若没有启动消息循环，则消息传递不可用）**