# ViewStub用法
* **控件ViewStub是一个不可见的、零尺寸的轻量级View，它可以在运行时进行延迟加载：即想加载的时候加载，不想加载的时候就不会初始化其中的布局，节省内存**；

* **一般使用方式：**

		1.在xml中添加要隐藏的布局
			android:id:用于设置ViewStub自身的id
			android:inflatedId:设置被映射的父布局的id，可用findViewById获取
			
		2.在主布局中添加ViewStub控件，并用layout属性引入布局
		3.调用.inflate()方法显示ViewStub
			注：映射布局的控件在ViewStub显示之前都不能调用findViewById来获取，会报空；


* **一般来讲ViewStub都是通过调用inflate()方法来加载被指定的父布局将自己替换为加载的布局，替换后，ViewStub将会从布局树中移除，因此inflate方法在一次刷新中只能使用一次，再次调用会报空指针的异常**
* **ViewStub也能被setVisibility方法启动，只不过存在区别：见inflate与Visibility的异同**

* **适用场景：如官方介绍所说，在需要的地方对ViewStub进行加载，不需要的地方不用加载，一般用于较大型的项目并且遇到大量的视图需要绘制的时候，此时可以使用ViewStub对网络申请的结果进行逻辑上的处理，相较于include、merge而言更灵活，比visibility也更节省内存**

# ViewStub方法：

* **(void)draw(Canvas canvas):**绘制方法
* **(void)onMeasure(int widthMeasureSpec, int heightMeasureSpec):绘制之前的测量方法**
* **(int)getInflatedId():**get需要加载的布局id
* **(LayoutInflater)getLayoutInflater():**get布局加载器
* **(int)getLayoutResource():**get需要加载的布局
* **(View)inflate():**见源码解析
* **(void)setInflatedId(int inflatedId):**对应android:inflatedId
* **(void)setLayoutInflater(LayoutInflater inflater):**设置布局加载器
* **(void)setLayoutResource(int layoutResource):**对应android:layout
* **(void)setOnInflateListener(ViewStub.OnInflateListener inflateListener):**ViewStub的监听方法，传入一个ViewStub内置的回调接口，当ViewStub调用inflate时，在布局扩张之前会调用这个方法，回调返回的该ViewStub实例和其引用的View实例，调用View的findViewById方法可以获得布局中的控件实例，虽然不知道有什么用，不过这个监听方法确实可以通过返回的View在布局**完全展示（？）** 之前拿到其中控件的实例，可以进行一些初始化操作；**注意：该监听方法应声明在inflate方法之前，如果在inflate方法之后，那么此时布局已经完成了扩充，将不会调用该方法**
* **(void)setVisibility(int visibility):**可以启动ViewStub，比inflate更加安全，见inflate与setVisibility的异同
# ViewStub优点与缺点

* **优点：由于只有被inflate()之后ViewStub才会被实例化，因此比View.GONE更加省内存,且其实例化的可选择性，因此比include、merge更加灵活；**
* **缺点：一次刷新中只能使用一次inflate方法，重复使用会报错**
* **不能使用merge**

# inflate与setVisibility的异同

首先可以看下ViewStub.setVisibility()的源码：

	public void setVisibility(int visibility) {
        if (mInflatedViewRef != null) {
            View view = mInflatedViewRef.get();
            if (view != null) {
                view.setVisibility(visibility);
            } else {
                throw new IllegalStateException("setVisibility called on un-referenced view");
            }
        } else {
			//关键
            super.setVisibility(visibility);
            if (visibility == VISIBLE || visibility == INVISIBLE) {
                inflate();
            }
        }
    }

网上的解释：首先是看弱引用中是否持有当前需要加载的view对象，如果有的话，改变一下他的显示、隐藏状态。如果没有的话，说明是第一次加载，当然需要走到inflate()方法中去

自己的理解：可以把mInflatedViewRef看作是一种ViewGroup类型，当调用ViewStub的setVisibility方法时，因为ViewStub引用的布局没有实例，所以mInflatedViewRef为null，所以直接进入inflate的判断；当ViewStub被inflate了之后再次调用这个方法，此时布局有了实例，就可以对View的视图进行操作了；

**即：当调用setVisibility方法时一样要走inflate流程，而且传入GONE参数时不会调用inflate，由于前面对多次调用的逻辑处理：如果View存在实例，那么就会调用View.setVisibility;如果View不存在实例，则会抛出异常，而不会让程序崩溃；所以我觉得setVisibility方法要比inflate方法更加灵活；**

**如果通过setVisibility来加载,那么通过判断可见性即可;如果通过inflate()来加载,判断ViewStub 的ID是否为null来判断(findViewById(…))**

**注：调用了setVisibility方法后直接再次调用inflate方法仍然会报错**  


* **View.setVisibility与ViewStub的区别：view的View.GONE等属性是看不到，但实际存在，存在实例，而ViewStub隐藏时包含的控件没有实例化**

# inclede、merge、ViewStub
* **include:**
	1. 什么是include：include就是在一个布局中，导入另一个布局；
	2. 为什么要使用include：相同的页面只需要写一次，在需要的地方include即可，提高了共通布局的复用性
	3. 怎么使用include：
		1. 定义共通布局
		2. include引入共通布局
		3. ！注：检测同时或分开引入两个相同布局时点击事件响应的逻辑
	4. 当使用两个相同的布局时，怎么区别：在include相同页面时指定不同id，通过id找到对应的View：

				<LinearLayout>
    			<include layout="@layout/include_layout"
        		android:id="@+id/include_lay_1"/>
				<include layout="@layout/include_layout"
        		android:id="@+id/include_lay_2"/>     
				</LinearLayout>

				View include1 = findViewById(R.id.include_lay_1);
				获取父布局下子控件的实例：include1.findViewById(...);
				LinearLayout include2 = findViewById(R.id.include_lay_2);


	5. 很明显，被include引入的布局是必须要加载的，（！检测）它的作用就是可以通过更改id的方式来重用相同的布局，但并不能起到节省内存的作用
	


* **merge:**  
	1.merge不是导入布局，而是作为一个通用的根布局被include引入，merge包含的子控件直接使用include的父布局以减少布局的层级，从而达到减少性能消耗的目的；  
	2.merge的用法：  
		如果include标签的父布局 和 include布局的根容器是相同类型的，那么根容器的可以使用merge代替。 

		<merge xmlns:android="http://schemas.android.com/apk/res/android">

		    <Button
		        android:id="@+id/ok"
		        android:layout_width="match_parent"
		        android:layout_height="wrap_content"
		        android:layout_marginLeft="20dp"
		        android:layout_marginRight="20dp"
		        android:text="OK" />
		
		    <Button
		        android:id="@+id/cancel"
		        android:layout_width="match_parent"
		        android:layout_height="wrap_content"
		        android:layout_marginLeft="20dp"
		        android:layout_marginRight="20dp"
		        android:layout_marginTop="10dp"
		        android:text="Cancel" />
		
		</merge>
		然后用include引入这个布局

	3.可以看出merge就像一个空的LinearLayout等基本布局，include引用的时候相当于忽略掉了merge，能起到优化性能的目的，但相比于ViewStub还是不够灵活，而且并不能直接被ViewStub引用,但是ViewStub引用的布局中含有include嵌套的merge是可行的；merge中可以嵌套ViewStub
	
* **特别注意：当include多次重用同一个merge布局时，通过更改include：id的方式是获取不到merge布局的根布局的，但是可以通过更改include父布局的id可以获取到merge布局的实例，侧面印证了merge=include父布局**

# 从ViewStub源码分析为何ViewStub能做到优化、节省内存

mInflatedId：需要加载的布局id  
mLayoutResource：需要加载的布局  
mInflatedViewRef：对需要加载的布局view的一个弱引用的持有  

	//绘制之前的测量方法
	@Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(0, 0);
    }
	
	//其中的一个构造方法
	public ViewStub(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context);
		。。。

        setVisibility(GONE);
        。。。
    }

**以上两个方法可以看出ViewStub为何节省内存：在onMeasure方法，只有一句代码setMeasuredDimension(0, 0);所以说，不管ViewStub设置多大，在初始化时都是一个没有大小、零尺寸的View，而且在其构造方法中，将visibility设置为GONE不可见，即两项综合在inflate之前是不加载实例的**


**inflate：过程解析**

	public View inflate() {
	        final ViewParent viewParent = getParent();
	
	        if (viewParent != null && viewParent instanceof ViewGroup) {
	            if (mLayoutResource != 0) {

	                final ViewGroup parent = (ViewGroup) viewParent;

					//注意下面两个方法
					//在inflate方法中首先会调用inflateViewNoAdd方法
	                final View view = inflateViewNoAdd(parent);

					//然后会调用replaceSelfWithView(view, parent)方法；
	                replaceSelfWithView(view, parent);
	
					//最后，将加载好的view放在弱引用中持有，用于后面的显示隐藏对view的重新操作，同时回调onInflate
	                mInflatedViewRef = new WeakReference<>(view);
	                if (mInflateListener != null) {
	                    mInflateListener.onInflate(this, view);
	                }
	
	                return view;
	            } else {
	                throw new IllegalArgumentException("ViewStub must have a valid layoutResource");
	            }
	        } else {
	            throw new IllegalStateException("ViewStub must have a non-null ViewGroup viewParent");
	        }
	    }


	//inflateViewNoAdd方法就是通过factory加载设置的视图mLayoutResource，并返回view对象。
	private View inflateViewNoAdd(ViewGroup parent) {
        final LayoutInflater factory;
        if (mInflater != null) {
            factory = mInflater;
        } else {
            factory = LayoutInflater.from(mContext);
        }
        final View view = factory.inflate(mLayoutResource, parent, false);

        if (mInflatedId != NO_ID) {
            view.setId(mInflatedId);
        }
        return view;
    }

	
	//将inflateViewNoAdd方法返回的view对象加入到ViewStub的父view上，完成ViewStub占位的替换
	private void replaceSelfWithView(View view, ViewGroup parent) {
        final int index = parent.indexOfChild(this);
        parent.removeViewInLayout(this);

        final ViewGroup.LayoutParams layoutParams = getLayoutParams();
        if (layoutParams != null) {
            parent.addView(view, index, layoutParams);
        } else {
            parent.addView(view, index);
        }
    }


	

