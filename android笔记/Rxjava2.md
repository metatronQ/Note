# 四个基本概念：  
* Observable：被观察者
* Obserber:观察者
* Subcribe：订阅
* Event：事件

# 三种事件：
* onNext：普通事件
* onCompleted：当事件队列中不在有新的onNext事件发出时，需要触发onCompleted事件作为标志
* onError：与onCompleted事件不能共存，并且是事件序列的最后一个

# 基本实现：

* **创建观察者-Observer**  


	
		Observer<String> observer = new Observer<String>() {
		    @Override
		    public void onCompleted() {
		        Log.d(TAG, "onCompleted");
		    }
		
		    @Override
		    public void onError(Throwable e) {
		        Log.d(TAG, "onError");
		    }
		
		    @Override
		    public void onNext(String s) {
		        Log.d(TAG, "onNext");
		    }
		};


除了Observer接口之外，RxJava还内置了一个实现了Observer的抽象类：**Subscriber**，它对Observer接口进行了一些扩展，实质上在RxJava的subscribe过程中，Observer也总是被转换成为一个Subscriber再使用，他们的区别在与：


1.onStart：这是新增的方法，它会在subscribe刚开始，而事件还未发送之前被调用，它总是在subscribe所发生的线程被调用。

2.unsubscribe：这是它实现的另一个接口Subscription的方法，用于取消订阅，在这个方法被调用后，Subscriber将不再接收事件，一般在调用这个方法前，可以使用isUnsubscribed判断一下状态，Observable在订阅之后会持有Subscriber的引用，因此不释放会有内存泄漏的危险。

* **创建被观察者-Observable**  

		Observable observable = Observable.create(new Observable.OnSubscribe<String>() {
		
		    @Override
		    public void call(Subscriber<? super String> subscriber) {
		        subscriber.onNext("Hello World!");
		        subscriber.onCompleted();
		    }
		});

这里传入了一个Observable.OnSubscribe<T>对象作为参数，它会被存储在返回的Observable对象当中，它的作用相当于一个计划表，当Observable被订阅的时候，OnSubscribe的call方法会自动被调用，事件序列被依次触发

*创建事件序列的方式:*  
1.create:常规方法  
2.快捷方法：  
just(T...)  

	Observable observable = Observable.just("Hello", "Hi", "Aloha");
	from(T[])/from(Iterable<? extends T>)

	String[] words = {"Hello", "Hi", "Aloha"};
	Observable observable = Observable.from(words);

* **订阅-subscribe**  
	
		observable.subscribe(observer);
		observable.subscribe(subscriber);

# 线程控制-Scheduler

* Schedulers.immediate:直接在当前线程运行
* Schedulers.newThread:总是启用新线程
* Schedulers.io:内部实现是一个无数量上限的线程池，可以重用空闲的线程，不要把计算工作放在io，避免创建不必要的线程
* Schedulers.computation:使用固定的线程池，大小为CPU核数
* AndroidSchedulers.main
* Thread:指定的操作将在Android主线程中运行


### 控制线程的两个方法：

subscribeOn:指定subscribe发生的线程，即Observable.OnSubscribe被激活所处的线程，也就是call方法执行时所处的线程（举例：申请网络请求）

observeOn:指定Subcriber(观察者)所运行在的线程（更新UI）

！ subscribeOn() 指定的是发送事件的线程, observeOn() 指定的是接收事件的线程.

# 变换：
将事件序列中的对象或整个序列进行加工处理，转换不同的事件或序列

* **map():**   
通过FuncX，参数中的Integer转换成String(Func1)

		observable.map(new Func1<Integer, String>() {
	
	    @Override
	    public String call(Integer integer) {
	        long mapId = Thread.currentThread().getId();
	        return "My Number is:" + integer;
	    }
		}).subscribeOn(Schedulers.io()).observeOn(AndroidSchedulers.mainThread()).subscribe(subscriber);

* **flatMap:**  
flatMap返回的是一个Observable对象，而且它并不直接把这个对象传给Subscriber，而是通过这个新建的Observable来发送事件

调用过程：  
 1.使用传入的事件对象创建一个Observable；  
 2.激活这个Observable，通过它来发送事件；  
 3.每一个创建出来的Observable发送的事件，被汇入同一个Observable，然后统一的交给Subscribe的回调方法；  

	
	observable.flatMap(new Func1<List<String>, Observable<String>>() {
	    @Override
	    public Observable<String> call(List<String> strings) {
	        return Observable.from(strings);
	    }
	}).subscribe(subscriber);


