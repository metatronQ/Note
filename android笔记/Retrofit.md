# 使用步骤：  

## 1.添加Retrofit库的依赖；

	//retrofit基于Okhttp
	dependencies {
	    compile 'com.squareup.retrofit2:retrofit:2.0.2'
	    // Retrofit库
	    compile 'com.squareup.okhttp3:okhttp:3.1.2'
	    // Okhttp库
	  }

	//添加网络权限
	<uses-permission android:name="android.permission.INTERNET"/>
  
## 2.创建接收服务器返回数据的类；

根据返回的数据的格式和数据解析方式（Json、xml等定义）：

	Reception.java

	public class Reception {
    
    }

## 3.创建用于描述网络请求的接口；

Retrofit将 Http请求 抽象成 Java接口：采用 注解 描述网络请求参数 和配置网络请求参数

	public interface GetRequest_Interface {

	    @GET("openapi.do?keyfrom=Yanzhikai&key=2032414398&type=data&doctype=json&version=1.1&q=car")
	    Call<Translation>  getCall();
	    // @GET注解的作用:采用Get方法发送网络请求
	 
	    // getCall() = 接收网络请求数据的方法
	    // 其中返回类型为Call<*>，*是接收数据的类（即上面定义的返回数据类）
	    // 如果想直接获得Responsebody中的内容，可以定义网络请求返回值为Call<ResponseBody>
	}

网络请求完整的Url = 在创建Retrofit实例时通过.baseURL()设置 + 网络请求接口的注解设置

## 4.创建Retrofit实例

	 Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("http://fanyi.youdao.com/") // 设置网络请求的Url地址
                .addConverterFactory(GsonConverterFactory.create()) // 设置数据解析器
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create()) // 支持RxJava平台
                .build();

关于数据解析器：  

* Retrofit支持多种数据解析方式
* 使用时需添加依赖

Gson：	com.squareup.retrofit2:converter-gson:2.0.2  
Jackson：	com.squareup.retrofit2:converter-jackson:2.0.2  
Simple： XML	com.squareup.retrofit2:converter-simplexml:2.0.2  
Protobuf：	com.squareup.retrofit2:converter-protobuf:2.0.2  
Moshi：	com.squareup.retrofit2:converter-moshi:2.0.2  
Wire：	com.squareup.retrofit2:converter-wire:2.0.2  
Scalars：	com.squareup.retrofit2:converter-scalars:2.0.2  

关于网络适配器（CallAdapter）
  
* Retrofit支持多种网络请求适配器方式：guava、Java8和rxjava
* 需添加依赖

guava：	com.squareup.retrofit2:adapter-guava:2.0.2  
Java8：	com.squareup.retrofit2:adapter-java8:2.0.2  
rxjava2：	com.squareup.retrofit2:adapter-rxjava2:2.0.2(Rxjava2)  


## 5.创建网络请求接口实例并配置网络请求参数

	// 创建 网络请求接口 的实例
    GetRequest_Interface request = retrofit.create(GetRequest_Interface.class);

    //对 发送请求 进行封装
    Call<Reception> call = request.getCall();



## 6.发送网络请求（同步/异步）

	//发送网络请求(异步)
        call.enqueue(new Callback<Translation>() {
			
			//？疑问：当我尝试将Retrofit请求封装成一个函数，并且有传一个参数Call<?>时，onResponse方法却不能传入Call<?>,并报'? extends com.example.demotest.T' cannot be instantiated directly的错，初步估计是泛型方面的问题

			//想要这么做的目的：想要任何一种格式的数据，只要传入类就可以完成解析，而没必要改函数

			//?同理：就算我不用带有通配符的泛型实例化，但是我创建一个T类，封装的函数传入T类型，这里也同样实例化一个T类型，依旧会报同样的错，不能理解

			//不能实例化带有通配符的泛型
					
            //请求成功时回调
            @Override
            public void onResponse(Call<Translation> call, Response<Translation> response) {
				
                //请求处理,输出结果
                response.body().show();
            }

            //请求失败时候的回调
            @Override
            public void onFailure(Call<Translation> call, Throwable throwable) {
                System.out.println("连接失败");
            }
	});

	// 发送网络请求（同步）
	Response<Reception> response = call.execute();
	

## 7.处理服务器返回的数据

* 通过response类的body()方法对返回你的数据进行处理

		reponse.body()[.show()//打印]



