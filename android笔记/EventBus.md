# 1.简介  
EventBus是一种用于Android的事件-订阅总线，简化了应用程序内各个组件之间进行通信的复杂度，尤其是碎片之间进行通信的问题，可以避免由于广播通信而带来的诸多不便。

## 1.1.三个角色  
* Event：事件，可以是任意类型（泛型），EvevtBus会根据事件的类型进行全局的通知（根据泛型的不同更改通知逻辑）。
* Subscriber：事件订阅者，定义事件处理的方法：3.0之前事件处理的方法必须是如下四种，3.0之后可以随意取，但必须加上注解@subscribe，并指定线程模型；

> 1.onEvent  
> 2.onEventMainThread  
> 3.onEventBackgroundThread  
> 4.onEventAsync  
  
> 其中，@subscribe共有三个参数
>> * threadMode:指定线程模型
>> * sticky：订阅黏性事件时指定
>> * priority：优先级指定

* Publisher：事件的发布者，可以在任意线程里发布事件。通过使用EventBus.getDefault() 可得到一个EventBus对象，在调用post(Object)方法发布。  


## 1.2.四种线程模型
threadMode

* POSTING：默认，表示事件处理函数的线程跟发布事件的线程在同一个线程；
* MAIN：表示事件处理函数的线程在主线程（UI）线程，因此不能进行耗时操作；
* BACKGROUND：表示事件处理函数的线程在后台线程，因此不能进行UI操作。如果发布事件的线程是主线程（UI线程），那么事件处理函数将会开启一个后台线程，如果发布事件的线程是在后台线程，那事件处理函数就会使用该线程。
* ASYNC：表示无论事件发布的线程是哪一个，事件处理函数始终会新建一个子线程运行，同样不能进行UI操作；


# 2.EventBus使用  

## 2.1 引入依赖
	implementation 'org.greenrobot:eventbus:3.1.1'

## 2.2 定义事件  
	public class MessageWrap{
		public final String message;
		public static MessageWrap getInstance(String message){
			return new MessageWrap(message);
		}

		private MessageWrap(String message){
			this.message = message;
		}
	}
## 2.3 发布事件  
	

	public class SubscriberActivity extends AppCompatActivity{
		@Override
		protected void onCreateView(Bundle savedInstanceState){
			super.onCreate(savedInstanceState);
        	setContentView(R.layout.activity_subscriber);
			//在本活动对事件进行订阅注册
			Button register = (Button)findViewById(R.id.register_btn);
			register.setOnClickListener(new View.OnClickListener() {
	            @Override
	            public void onClick(View v) {
	                EventBus.getDefault().register(SubscriberActivity.this);  
	
	                //在View.onClickListener（）中的this不是指的是当前的Activity，而指的是监听View
	
	                Toast.makeText(SubscriberActivity.this, "注册订阅者", Toast.LENGTH_SHORT).show();
	
		            }
	        });
			//跳转至另一个活动
			Button intentPub = (Button)findViewById(R.id.intent_publish);
			intentPub.setOnClickListener(new View.OnClickListener() {
	            @Override
	            public void onClick(View v) {
	                Intent intent = new Intent(SubscriberActivity.this,SendActivity.class);
	                startActivity(intent);
	            }
	        });
		}
		@Override
		protected void onDestroy(){
			super.onDestroy();
			EventBus.getDefault().unregister(this);
		}
		
		@Subscribe(threadMode = ThreadMode.MAIN)
		public void onGetMessage(MessageWrap messageWrap){
			//接收处理事件
			Log.d("SubscriberActivity", ""+messageWrap.message);
		}

	}

	//另一个活动发布事件
	public class SendActivity extends AppCompatActivity {
	    EditText editText;
		   @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_send);
	        editText = (EditText)findViewById(R.id.edit_text);
	        Button sendMessage = (Button)findViewById(R.id.publisher);
	        sendMessage.setOnClickListener(new View.OnClickListener() {
	            @Override
	            public void onClick(View v) {
	                publishContent();
	            }
	        });
    }
		//发送事件
	    public void publishContent(){
	        String msg = editText.getText().toString();
	        EventBus.getDefault().post(MessageWrap.getInstance((msg)));
	        Toast.makeText(this,"Publish!" + msg,Toast.LENGTH_SHORT).show();
	    }
	}
## 2.4 黏性事件：Sticky
指发送了该事件后，订阅者依然能够接收到的事件:
因为请求接口然后再发送事件再进行控件的更新。有时候该控件所在的页面可能没有初始化好。这时候eventbus所发送的事件就不会起作用。这时候就要用到粘性事件。  

* 粘性事件可以先发送事件，待接收方订阅后接受事件    

		//在Activity1中对黏性事件进行订阅注册
		@Subscribe(threadMode = ThreadMode.MAIN, sticky = true)
		public void onGetStickyEvent(MessageWrap message) {
		    //处理事件
		}
		
		//在Activity2中调用EventBus的postSticky方法发布黏性事件
		private void publishStickyontent() {
		    String msg = 
		    EventBus.getDefault().postSticky(MessageWrap.getInstance(msg));
		}
## 2.5 优先级 
* priority: int 值越大的优先级高，会先接收到事件；

* 注：  
1.优先级只在相同的ThreadMode中起作用  
2.只有当ThreadMode为POSTING的时候，才能停止事件的继续分发  
  
		// 用来判断是否需要停止事件的继续分发
		private boolean stopDelivery = false;

		Button stopSend = (Button)findViewById(R.id.stop_send);
		stopSend.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                stopDelivery = true;
            }
        });

		@Subscribe(threadMode = ThreadMode.POSTING,priority = 0)
	    public void onGetMessage(MessageWrap messageWrap){
	        Log.d("SubscriberActivity", ""+messageWrap.message);
	    }
	
	    @Subscribe(threadMode = ThreadMode.POSTING,sticky = true,priority = 1)
	    public void onGetStickyMessage(MessageWrap messageWrap){
	        Log.d("SubscriberActivity", ""+messageWrap.message + " Sticky");
	        if (stopDelivery){
	            EventBus.getDefault().cancelEventDelivery(messageWrap);
	        }
	    }