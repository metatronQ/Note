# DataBinding简介：

DataBinding是2015年GoogleI/O大会上介绍的一个数据绑定框架，主要用于解决两个问题：

* 需要多次使用findViewById，损害了应用性能且令人厌烦
* 更新UI数据需切换至UI线程，将数据分解映射到各个View比较麻烦（虽然可以用handler等，但handler内部其实也实现了线程切换）


# DataBinding基本使用：

	android{
		...
		dataBinding{
			enabled true
		}
	}


* 单纯的摆脱findViewById:布局以layout作为根布局，根布局下的ViewGroup为要展示的布局，即在原来布局的基础上加一层<layout>标签  

		//xml
		<layout xmlns:android = "http://schemas.android.com/apk/res/android">
		    <LinearLayout
		        android:layout_width="match_parent"
		        android:layout_height="match_parent"
		        android:orientation="vertical">
		        <Button
					//控件的id作为获得实例的凭依
		            android:id="@+id/button"
		            android:layout_width="match_parent"
		            android:layout_height="wrap_content"
		            android:text="不用findViewById的按钮"/>
		    </LinearLayout>
		</layout>

		//活动中：通过DataBinding加载布局都会相应的生成Binding对象，对象名为布局文件名加上后缀Binding,此处通过DataBindingUtil的setContentView方法得到生成对象的实例，这里可以看一下它的源码
		ActivityMainBinding activityMainBinding = DataBindingUtil.setContentView(this,R.layout.activity_main);
        
        //通过binding.控件id的形式获得控件实例
        activityMainBinding.button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Log.d(TAG, "onClick: 获取到实例并设置点击监听");
            }
        });

* 绑定基本数据类型及String

		xml
		//需要绑定的数据被data标签包含
		<data>
			//variable:变量 name：类比java中的对象 type：类比java中的各种类型
	        <variable
	            name="content"
	            type="String" />
	
	        <variable
	            name="enabled"
	            type="boolean" />
	    </data>

		<Button
            android:id="@+id/button1"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"

			//单向绑定使用@{name（基本数据类型及String）}
            android:enabled="@{enabled}"
            android:text="@{content}"/>

		活动
		//2
		//在xml文件中绑定变量之后，编译会生成相应的setter和getter方法用于对控件的属性进行设置
        activityMainBinding.setContent("绑定String的按钮");
        activityMainBinding.setEnabled(false);
        activityMainBinding.button1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Log.d(TAG, "onClick: 得不到的点击");
            }
        });

* 绑定Model（Model此处指模型，或者对应java中的对象）

		//假设此时已经创好了一个只包含一个text（String）属性的User类，类中含有该属性的setter、getter方法
		
		//此时的类型为类所在的包的路径的对象类型，当然也可以在外导包，type只需要为类名即可
		<variable
            name="User"
            type="com.example.mvvmtest.User" />

		//控件中绑定采用 @{name.类中的属性或方法名}
		 android:text="@{User.name}"

		//活动
		//调用生成的setter方法设置属性
		activityMainBinding.setUser(new User("张三","男"));
		
* 绑定事件：
		
		//首先先创建监听的接口，接口中包含点击事件的集合
		//简单起见，只创建一个点击事件
		public interface Click {
		    public void click1(String s);
		}
		
		xml
		<variable
            name="event"
            type="com.example.mvvmtest.Click" />
        <variable
            name="title"
            type="String" />
        <variable
            name="title1"
            type="String" />

		//onClick的三种绑定方式
		//android:onClick="@{event.click1}" event.click1 = (view)->event.click1(view)
		//android:onClick="@{event::click1}"
		//android:onClick="@{()->event.cilck1(title1)}"
		
		<Button
            android:id="@+id/button3"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:onClick="@{()->event.click1(title1)}"
            android:text="@{title}"/>

		//活动
		activityMainBinding.setTitle("事件按钮");
        activityMainBinding.setTitle1("按下了按钮");
        activityMainBinding.setEvent(new Click() {
            @Override
            public void click1(String s) {
                activityMainBinding.setTitle(s);
            }
        });

* 通过静态方法转换数据类型（类似java调用静态方法，不需要在活动中调用）

		//静态方法的类
		public class Utils {
		    public static String getName(Object o){
		        return o.getClass().getName();
		    }
		}

		//调用静态方法，需要先导入需要的包（可以使用java自带的包，且lang包不需要导包）
		<import type="com.example.mvvmtest.Utils"/>

		<Button
            android:id="@+id/button4"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{Utils.getName(User)}"/>
		

* 通过运算符操作数据

	DataBinding支持的表达式有：
	
	数学表达式： + - / * %
	
	字符串拼接 +
	
	逻辑表达式 && ||
	
	位操作符 & | ^
	
	一元操作符 + - ! ~
	
	位移操作符 >> >>> <<
	
	比较操作符 == > < >= <=
	
	instanceof
	
	分组操作符 ()
	
	字面量 - character, String, numeric, null
	
	强转、方法调用
	
	字段访问
	
	数组访问 []
	
	三元操作符 ?:
	
	聚合判断（Null Coalescing Operator）语法 ‘??’：  
	
  		<!-- android:text="@{user2.content ?? @string/app_name}" 等价于 android:text="@{user2.content!=null? user2.content : @string/app_name}"-->


		

* 自定义Binding的类名：三种方式  
	1.通过指定全名的方式

		<data class="www.zhang.com.databinding.activity.Item">
		    ...
		</data>  
		
		import www.zhang.com.databinding.activity.Item;

	2.直接生成在项目的包目录下
		
		<data class=".Item">
		    ...
		</data>
		
		import www.zhang.com.databinding.Item;
	3.Activity在包的子目录下：

		<data class="Item">
		    ...
		</data>
		
		import www.zhang.com.databinding.databinding.Item;

* 绑定相同Model的操作:  
	1.同类的两个对象：  


		设置不同的对象名即可
		

	2.不同类的两个类，但类名相同：
		
		<import type="com.example.mvvmtest.User" />
        <import type="com.example.mvvmtest.UI.User" />
		
		<variable
            name="User"
            type="User" />

        <variable
            name="User1"
            type="User" />

		//出现上述情况的话，系统会分辨不出User类是属于哪个导入的包，默认User都属于最后导入的包；
		//此时就可以通过alias属性对不同的包进行重命名
		<import type="com.example.mvvmtest.UI.User" alias = "UI"/>

		<variable
            name="User1"
            type="UI" />
		活动中获取该User对象需要写全包名类型声明

* model变量改变自动更新数据

		Model类继承BaseObservable,BaseObservable实现android.databinding.Observable接口，Observable接口可以允许附加一个监听器到model对象以便监听对象上的所有属性的变化

		public class Person extends BaseObservable {
		    String name;
		    String age;
		    public Person(String name,String age){
		        this.name = name;
		        this.age = age;
		    }

			Observable接口有一个机制来添加和删除监听器，但通知与否由开发人员管理。为了使开发更容易，BaseObservable实现了监听器注册机制。DataBinding实现类依然负责通知当属性改变时。这是通过指定一个Bindable注解给getter以及setter内通知来完成的。
			
			设置这个注解后，会在BR类为当前属性添加一个字段值，用来通知改变
		    @Bindable
		    public String getName() {
		        return name;
		    }
		    public void setName(String name) {
		        this.name = name;
		        notifyPropertyChanged(BR.name);
		    }
		    @Bindable
		    public String getAge() {
		        return age;
		    }
		    public void setAge(String age) {
		        this.age = age;
		        notifyPropertyChanged(BR.age);
		    }
		}


   * notifyPropertyChanged(BR.参数名)通知更新这一个参数，需要与@Bindable注解配合使用。
   * notifyChange()通知更新所有参数，可以不用和@Bindable注解配合使用


* 绑定List/Map等集合数据

	支持List/Map集合类型绑定

    **值得注意的是：<variable>标签中不识别'<' 符号，因此表示<>符号时需使用转义字符&lt;**

* Observable自动更新
	
	Observable是一个接口，它的子类有BaseObservable,ObservableField,ObservableBoolean, ObservableByte, ObservableChar, ObservableShort, ObservableInt, ObservableLong, ObservableFloat, ObservableDouble, and ObservableParcelable，ObservableArrayList,ObservableArrayMap

	用上述类型取得的实例可以不用注解绑定就可以实现自动通知的效果


* DataBinding与include标签的结合

	引用的布局也遵循DataBinding结构，可以在子布局和父布局设置同一个变量对象，方便对引用的子布局进行设置

				<?xml version="1.0" encoding="utf-8"?><!--布局以layout作为根布局-->
			<layout>
			
			    <data >
			        <import type="www.zhang.com.databinding.model.Content"/>
			        <variable
			            name="con"
			            type="Content"/>
			    </data>
			    <!--我们需要展示的布局-->
			    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
			        xmlns:bind="http://schemas.android.com/apk/res-auto"
			        android:layout_width="match_parent"
			        android:layout_height="match_parent"
			        android:orientation="vertical">
			        <include
			            android:id="@+id/toolbar"
			            layout="@layout/toolbar"
			            android:layout_height="56dp"
			            android:layout_width="match_parent"
			            bind:content="@{con}" />
			 <!--通过命名空间将写有toolbar的xml文件中定义的content对象作为属性绑定con对象，这2个对象是同一个类-->
			            <TextView
			                android:text="@string/app_name"
			                android:layout_width="match_parent"
			                android:layout_height="wrap_content" />
			    </LinearLayout>
			</layout>

			<?xml version="1.0" encoding="utf-8"?>
			<layout >
			    <data>
			        <import type="www.zhang.com.databinding.model.Content"/>
			      <variable
			          name="content"
			          type="Content"/>
			    </data>
			
			<android.support.v7.widget.Toolbar
			    xmlns:android="http://schemas.android.com/apk/res/android"
			    android:id="@+id/toolbar"
			    xmlns:app="http://schemas.android.com/apk/res-auto"
			    android:layout_height="56dp"
			    android:layout_width="match_parent"
			    app:title="@{content.title}"
			    app:subtitle="@{content.subTitle}"
			    android:background="@color/colorPrimary"
			    app:titleTextColor="@android:color/white"
			    app:subtitleTextColor="@android:color/white" />
			</layout>

				Content con = new Content("Title","SubTitle");
				binding.setCon(con);
				
				//        binding.toolbar.setContent(con);  //这个测试没有效果，不会显示toolbar的title/subTitle，因为此处设置的是include的content属性
				//        binding.toolbar.toolbar.setTitle(""); 
				//        binding.toolbar.toolbar.setSubtitle("");
				
				        //下面的代码也可以通过DataBinding绑定数据
				        binding.toolbar.toolbar.setNavigationIcon(R.mipmap.ic_launcher);
				        setSupportActionBar(binding.toolbar.toolbar);
				        binding.toolbar.toolbar.setNavigationOnClickListener(new View.OnClickListener() {
				            @Override
				            public void onClick(View v) {
				                finish();
				            }
				        });

			


* DataBinding与RecyclerView的结合  
[DataBinding与RecyclerView](https://blog.csdn.net/qq_33689414/article/details/52205734)

注意：holder.getBinding().executePendingBindings()中的executePendingBindings方法


# 适配器基本使用
初步理解，可以认为这一个注解就是生成一个全局的控件属性。这一个注解面向一个public static方法，方法名自己定义。它注解的方法的第一个形参就是我们想要对其创建属性的控件类，第二个形参是赋给这一个自定义属性的值，因为是全局（public static）的，所以定义在何处都无所谓，一般创建一个Adapter类来容纳它：

	@BindingAdapter("android:paddingLeft")
	public static void setPaddingLeft(View view, int padding) {
	  view.setPadding(padding,
	                  view.getPaddingTop(),
	                  view.getPaddingRight(),
	                  view.getPaddingBottom());
	}

对相应的属性使用绑定表达式，在活动中为该属性设置值时就会自动寻找到上方被注解的方法进行set，然后在方法内部进行一系列的操作：包括可以设置监听，比较新旧值等；

DataBinding的@BindingAdapter也支持同时指定多个属性。默认来说，当指定多个属性时，这些属性要同时地使用到同一个控件上，有点像控件中强制要求设定属性layout_width和layout_height一样，否则的话会报错。我们也可以通过设置requireAll的值来指定是否强制所有的属性同时使用：
  
	@BindingAdapter({"imageUrl", "error"})
	public static void loadImage(ImageView view, String url, Drawable error) {
	  Picasso.get().load(url).error(error).into(view);
	}

	
可以在布局中使用适配器，如下面的示例所示。请注意，@drawable/venueError引用了应用程序中的资源。在资源周围加上@{}使其成为有效的绑定表达式。
此时是将venue中的inageUrl值直接赋值给了自定义的imageUrl属性，自动调用了自定义的方法

	<ImageView app:imageUrl="@{venue.imageUrl}" app:error="@{@drawable/venueError}" />

# 与ViewModel

# 双向数据绑定



