#### JakeWharton大神开源的一款Android视图的字段和方法绑定快速注解框架.也是Android开发中比较常用的一款快速注解框架了,可以不用不断的重复findViewById,在各种场合下快速绑定view的多种事件,大大提高了开发的效率

## 优点：
1、强大的View绑定和Click事件处理功能，简化代码，提升开发效率  
2、方便的处理Adapter里的ViewHolder绑定问题  
3、运行时不会影响APP效率，使用配置方便  
4、代码清晰，可读性强  
5、............   

## 配置：
	//项目build.gradle中
	repositories {
        google()
        mavenCentral()
    }
	dependencies {
        classpath 'com.android.tools.build:gradle:3.2.0'
        classpath 'com.jakewharton:butterknife-gradle-plugin:9.0.0'	
    }
	//APP build.gradle中
	android {
	    compileOptions {
	        sourceCompatibility 1.8
	        targetCompatibility 1.8
	    }
	...
	}
	//依赖；注：10版本以后只支持AndroidX，不兼容support
	dependencies {
    implementation 'com.jakewharton:butterknife:9.0.0'
    annotationProcessor 'com.jakewharton:butterknife-compiler:9.0.0'}

## ButterKnife的基本使用详解：

* 在Activity中绑定ButterKnife:在onCreate（）外注解绑定，在onCreate（）内bind（）绑定  

		public class MainActivity extends AppCompatActivity {
			@BindView(R.id.butter)
		    Button button;
		
		
		    @Override
		    protected void onCreate(Bundle savedInstanceState) {
		        super.onCreate(savedInstanceState);
		        setContentView(R.layout.activity_main);
		        ButterKnife.bind(this);
		    }
		
		    @OnClick(R.id.butter)
		    public void onClick(){
		        Log.e("111111","11111");
		        Toast.makeText(this, "绑定单个view事件", Toast.LENGTH_SHORT).show();
		    }
		}
		//ButterKnife.bind(this)必须在初始化绑定布局文件之后,否则会报错

* 在Fragment中绑定ButterKnife:在Fragment中需要在视图销毁时解绑Butterknife,否则会造成内存泄漏.
	
		public class ExampleFragment extends Fragment {
	
		    private Unbinder unbinder;
		    @BindView(R.id.example)
		    Button example;
		
		    @Nullable
		    @Override
		    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
		       View view = View.inflate(getContext(),R.layout.fragment_example,null);
		        unbinder = ButterKnife.bind(this,view);
		        return view;
		
		    }
		
		    @Override
		    public void onDestroyView() {
		        super.onDestroyView();
		        unbinder.unbind();//视图销毁时必须解绑
		    }
		}	

* 在Adapter的ViewHolder中绑定Butterknife:在Adapter的ViewHolder中使用，将ViewHolder加一个构造方法，在new ViewHolder的时候把view传递进去。使用ButterKnife.bind(this, view)进行绑定
		
		@NonNull
		@Override
		public MyViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
		        View itemView = LayoutInflater.from(parent.getContext()).inflate(R.layout.recy_dynamic_state_item,parent,false);
		        MyViewHolder myViewHolder = new MyViewHolder(itemView);//此处将view传入
		        itemView.setOnClickListener(this);
		        return myViewHolder;
			}
		
			public class MyViewHolder extends RecyclerView.ViewHolder {
		        @BindView(R.id.iv_photo)
		        SimpleDraweeView simpleDraweeView;
		        @BindView(R.id.tv_title)
		        TextView tvTitle;
		        @BindView(R.id.tv_detail)
		        TextView tvDetail;
		        @BindView(R.id.date)
		        TextView date;
		        @BindView(R.id.avatar_user)
		        SimpleDraweeView avatarUser;
		        @BindView(R.id.username)
		        TextView userName;
		
		        public MyViewHolder(View itemView) {
		             super(itemView);
		             ButterKnife.bind(this,itemView);//此处进行绑定
			}
		}	

## 更多注解方式： 
	@BindView—->绑定一个view；id为一个view 变量
	
	@BindViews —-> 绑定多个view；id为一个view的list变量
	
	@BindArray—-> 绑定string里面array数组；@BindArray(R.array.city ) String[] citys ;
	
	@BindBitmap—->绑定图片资源为Bitmap；@BindBitmap( R.mipmap.wifi ) Bitmap bitmap;
	
	@BindBool —->绑定boolean值
	
	@BindColor —->绑定color；@BindColor(R.color.colorAccent) int black;
	
	@BindDimen —->绑定Dimen；@BindDimen(R.dimen.borth_width) int mBorderWidth;
	
	@BindDrawable —-> 绑定Drawable；@BindDrawable(R.drawable.test_pic) Drawable mTestPic;
	
	@BindFloat —->绑定float
	
	@BindInt —->绑定int
	
	@BindString —->绑定一个String id为一个String变量；@BindString( R.string.app_name ) String meg;

## 更多事件注解：

	@OnClick—->点击事件
	
	@OnCheckedChanged —->选中，取消选中
	
	@OnEditorAction —->软键盘的功能键
	
	@OnFocusChange —->焦点改变
	
	@OnItemClick item—->被点击(注意这里有坑，如果item里面有Button等这些有点击的控件事件的，需要设置这些控件属性focusable为false)
	
	@OnItemLongClick item—->长按(返回真可以拦截onItemClick)
	
	@OnItemSelected —->item被选择事件
	
	@OnLongClick —->长按事件
	
	@OnPageChange —->页面改变事件
	
	@OnTextChanged —->EditText里面的文本变化事件
	
	@OnTouch —->触摸事件
	
	@Optional —->选择性注入，如果当前对象不存在，就会抛出一个异常，为了压制这个异常，可以在变量或者方法上加入一下注解,让注入变成选择性的,如果目标View存在,则注入, 不存在,则什么事情都不做

## Action接口与Setter接口：
* Action接口主要是为了对View或者Views进行管理初始化等操作，而Setter接口其实就是对view或者views的属性或者值进行操作.使用 ButterKnife.apply()方法启用接口.

		ButterKnife.Action<View> action = new ButterKnife.Action<View>() {
			@Override
			public void apply(@NonNull View view, int index) {
			     if (view instanceof Button ){
			             Button button = (Button) view;
			             button.setText("点击我");
			          }
			      }
			  };
			
		ButterKnife.Setter<View,Boolean> setter = new ButterKnife.Setter<View, Boolean>() {
			@Override
			    public void set(@NonNull View view, Boolean value, int index) {
			            view.setEnabled(value);
			        }
			};
			
			
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			     super.onCreate(savedInstanceState);
			     setContentView(R.layout.activity_main);
			     ButterKnife.bind(this);
			     ButterKnife.apply(buttons,action);//初始化或修改每个view的属性值
			     ButterKnife.apply(buttons,setter,true);
			} 

* apply()方法的实现原理：
	* action:apply接收两个参数：由外部指定的List泛型集合和一个自定义的action参数，方法中调用了ViewCollections的run方法，对List的集合进行逐个初始化
	* setter：apply接收三个参数：由外部指定的List泛型集合和一个自定义的setter参数以及由setter第二个泛型指定类型的value，方法调用了ViewCollections的set方法，对List集合进行逐个操作；
	* 由于是从集合中逐个取出view来进行操作且获得action和setter对象实际上是分别对两个接口中方法的实现，所以List集合可以不为同一view而且对view属性或值的操作更为便捷；