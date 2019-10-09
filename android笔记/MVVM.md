# MVVM基本介绍：
MVVM 软件设计模式由微软在2005年提出，下图及介绍总结自微软The MVVM Pattern和Implementing the MVVM Pattern。MVVM架构与微软自家产品关联性很强，并很适用于Android，这里仅仅是介绍MVVM模式的概念及MVVM模式中各模块所承担的职责。**MVVM架构其实跟MVP架构差不多，都是为了优化代码结构，降低数据请求与UI逻辑的耦合性而设计的，但是又有不同的地方**。

# 微软的MVVM四大结构:

* **View层:**就像在MVC和MVP模式中一样，视图是用户在屏幕上看到的结构、布局和外观（UI），决定如何呈现数据;
* **ViewModel层:**封装了View的显示逻辑和数据。不直接引用View。ViewModel实现来自View的命令（如点击事件）、处理（转换/聚合）View所需绑定的数据、通知View数据或状态的改变。ViewModel和数据和状态提供给View，但View决定了如何呈现。这里其实也算是与另外两种架构的区别之一；
* **Model层：**封装了业务逻辑和数据（业务逻辑是指所有有关数据检索与处理的程序逻辑），并且保证数据的一致性和有效性。为了最大化重用机会，Model不应包含任何用于特定ViewModel的处理逻辑。
* **Binder 绑定器：（这是MVVM架构模式与另两种模式的最大区别）**数据绑定技术的实现在MVVM中是必须的。Binder确保ViewModel中数据发生变化时能够及时通知View，使View呈现最新的数据；

# Google官方MVVM的四大结构：

[官方架构图]()

* **View**  
	显而易见 Activity/Fragment 便是MVVM中的View，当收到ViewModel传递来的数据时，Activity/Fragment负责将数据以你喜欢的方式显示出来。实际是View成还包括ViewDataBinding(根据xml自动生成)，上面中并没有体现。

* **ViewModel**  
	ViewModel作为Activity/Fragment与其他组件的连接器。负责转换和聚合Model中返回的数据，使这些数据易于显示，并把这些数据改变及时的通知给Activity/Fragment。ViewModel是具有生命周期意识的，当Activity/Fragment销毁时ViewModel的onClear方法会被回调，你可以在这里做一些清理工作。LiveData是具有生命周期意识的一个可观察的的数据持有者，ViewModel中的数据由LiveData持有，并且只有当Activity/Fragment处于活动时才会通知UI数据的改变，避免无用的刷新UI；

* **Model**  
	Repository及其下方就是Model了。Repository负责提取和处理数据。数据可以来自本地数据库(Room)，也可以来自网络，这些数据统一有Repository处理，对应隐藏数据来源及获取方式

* **Binder 绑定器(即数据绑定库DataBinding)**  
	上图中并没有标出绑定器在哪里，其实在任何MVVM的实现中，数据绑定技术都是必须的。而上图仅仅是应用架构图。
	Android中的数据绑定技术由 DataBinding和LiveData共同实现。当Activity/Fragment接收到来自ViewModel中的新数据时(由LiveData自动通知数据的改变)，将这些数据通过DataBinding绑定到ViewDataBinding中，UI将会自动刷新，而不用书写类似setText的方法。



# MVVM与MVP的对比
Model-View-Presenter（MVP），即模型-视图-表示层，架构被广泛应用于 Android 应用程序，通过引入表示层将视图与表示逻辑和模型分离。Model-View-ViewModel（MVVM），即模型-视图-视图模型，与 MVP 非常相似，视图模型充当增强的表示层，使用数据绑定器保持视图模型和视图同步。通过将视图绑定到视图模型属性上，数据绑定程序可以处理视图更新而无需手动更改数据来设置视图（例如，不用再设置控件 TextView 的setTest() 或者 setVisibility() 属性）。与 MVP 中的表示层一样，视图模型可以很容易地进行单元测试。本文介绍了数据绑定库和 MVVM 架构模式，以及它们在 Android 上协同工作方式。**两者最大的区别还是MVVM采用了数据绑定库绑定数据**


# 数据绑定库dataBinding的介绍：

## 什么是数据绑定：

数据绑定是一种把数据绑定到用户界面元素（控件）的通用机制。通常，数据绑定会将数据从本地存储或者网络绑定到显示层，其特征是数据的改变会自动在数据源和用户界面之间同步。

## 数据绑定库的好处：

	TextView textView = (TextView) findViewById(R.id.label);
	EditText editText = (EditText) findViewById(R.id.userinput);
	ProgressBar progressBar = (ProgressBar) findViewById(R.id.progress);
	 
	editText.addTextChangedListener(new TextWatcher() {
	   @Override public void beforeTextChanged(CharSequence s, int start, int count, int after) { }
	   @Override public void afterTextChanged(Editable s) { }
	   @Override public void onTextChanged(CharSequence s, int start, int before, int count) {
	       model.setText(s.toString());
	   }
	});
 
	textView.setText(model.getLabel());
	progressBar.setVisibility(View.GONE);


如上代码，大量的 findViewById() 调用之后，又是一大堆 setter/listener 之类的调用。 即使使用 ButterKnife 注入库也没有使情况改善。而数据绑定库就能很好地解决这个问题。

在编译时创建一个绑定类，它为所有视图提供一个 ID 字段，因此不再需要调用 findViewById() 方法。实际上，这种方式比调用 findViewById() 方法快数倍，因为数据绑定库创建代码仅需要遍历视图结构一次。

绑定类中也实现了视图文件的绑定逻辑，因此所有 setter 会在绑定类中被调用，你无须为之操心。总之，它能让你的代码变得更简洁。

## 如何设置数据绑定：

**1.**首先必须添加DataBinding支持，之后构建系统会收到提示对数据绑定启用附加处理，如，从布局文件创建绑定类。

	android {
   		...
	   dataBinding {
	       enabled = true
	   }
	   ...
	}

**2.**接下来，在 <layout> 标签中包装下布局中的顶层元素，以便为此布局创建绑定类。绑定类具有和布局 xml 文件相同的名称，只是在结尾添加 Binding，例如， Activity_main.xml 的绑定类名字是 ActivityMainBinding。 如上所示，命名空间的声明也移到布局标记中。然后，在布局标记内声明将需要绑定的数据作为变量，并设置好名称和类型。示例中，唯一的变量是视图模型，但后续变量会增加。你可以选择导入类，以便能使用 View.VISIBLE 或静态方法等常量。

	<TextView
	    android:id="@+id/my_layout"
	    android:layout_width="wrap_content"
	    android:layout_height="wrap_content"
	    android:visibility="@{vm.visible ? View.VISIBLE : View.GONE}">
	    android:padding="@{vm.bigPadding ? @dimen/paddingBig : @dimen/paddingNormal}"
	    android:text='@{vm.text ?? @string/defaultText + "Additional text."}' />

视图属性上的数据绑定指令以@开头，以大括号结束。你可以使用任何变量在数据段中导入你之前声明的变量。这些表达式基本支持你在代码中的所有操作，例如算术运算符或字符串连接。

Visibility 属性中还支持 if-then-else 三元运算符。还提供了合并运算符 ??，如果左边的值为空，则返回右操作数。在上述代码中，你可以像在正常布局中一样访问资源，因此你可以根据布尔变量的取值选择不同的 dimension 资源，也可以使用 padding 属性查看这些资源。

即使你在代码中使用 getters 和 setters，你所声明的变量的属性也可以用字段访问语法的形式访问。你可以在 slide 上的文本属性中看到此部分，其中 vm.text 调用视图模型的 getText() 方法。最后，也有一些小的限制，例如，不能创建新对象，但是数据绑定库仍然非常强大。

### 哪些属性是可以绑定的：

实际上，标准视图的大多数属性已经被数据绑定库支持。在数据绑定库内部，当你使用数据绑定时，库按照视图类型查找属性名称的 setter。例如，当你把数据绑定到 text 属性时，绑定库会在视图类中使用合适的参数类型查找 setText() 方法，上述示例是 String。

当没有对应的布局属性时，你也可以使用数据绑定的 setter。例如，你可以在 xml 布局中的 recycleler 视图上使用 app:adapter 属性，以利用数据绑定设置适配器参数。

对于标准属性，不是所有的都在 View 上有对应的 setter 方法。例如，paddingLeft 情况下，数据绑定库支持自定义的 setter，以便将绑定转移到 padding 属性上。但是，遇到 layout_marginBottom 的情况，当绑定库没有提供自定义 setter 时我们要怎么处理呢？

### 自定义setter

对于上述情况，自定义 setter 可以被重写。Setter 是使用 @BindingAdapter 注解来实现的，布局属性使用参数命名，使得绑定适配器被调用。上面示例提供了一个用于绑定layout_ marginBottom的适配器。
方法必须是 public static void，而且必须接受绑定适配器调用的首个视图类型作为参数，然后将数据强绑定到你需要的类型。在这个例子中，我们使用一个 int 类型和类型 View（子类型）定义一个绑定适配器。最后，实现绑定适配器接口。对于layout_ marginBottom，我们需要获取布局参数，并且设置底部间隔：

	@BindingAdapter("android:layout_marginBottom")
	public static void setLayoutMarginBottom(View v, int bottomMargin) {
	   ViewGroup.MarginLayoutParams layoutParams =
	           (ViewGroup.MarginLayoutParams) v.getLayoutParams();
	  
	   if (layoutParams != null) {
	       layoutParams.bottomMargin = bottomMargin;
	   }
	}


也可能需要设置多种属性以绑定适配器调用。为了达到此目的，MMVM 会提供你的属性名称列表并用于 @BindingAdapter 实现注解。另外，在现有方法中，每个属性都有自己的名称。只有在所有声明的属性被设置后，这些 BindingAdapter 才会被调用。

在加载图片过程中，我想为加载图片定义一个绑定适配器来绑定 URL 与 placeHolder。如你所见，通过使用 Picasso image loading library，绑定适配器非常容易实现。你可以在自定义绑定适配器中使用任何你想要的方法。

	@BindingAdapter({"imageUrl", "placeholder"})
	public static void setImageFromUrl(ImageView v, String url, int drawableId) {
	   Picasso.with(v.getContext().getApplicationContext())
	           .load(url)
	           .placeholder(drawableId)
	           .into(v);
	}


**3.在代码中使用绑定:**


现在我们在 xml 文件中定义了绑定，并且编写了自定义 setter，那我们如何在代码中使用绑定呢？ 数据绑定库通过生成绑定类为我们完成所有的工作。要获取布局的相应绑定类的实例，就要用到库提供的辅助方法。Activity 对应使用 DataBindingUtil.setContentView()，fragment 对应使用 inflate()，视图拥有者请使用 bind()。 如前所述，绑定类为定义 final 字段的 ID 提供了所有视图。同样，您可以在绑定对象的布局文件中设置你所声明的变量。

	MyBinding binding;
 
	// For Activity
	binding = DataBindingUtil.setContentView(this, R.layout.layout);
	// For Fragment
	binding = DataBindingUtil.inflate(inflater, R.layout.layout, container, false);
	// For ViewHolder
	binding = DataBindingUtil.bind(view);
	 
	// Access the View with ID text_view
	binding.textView.setText(R.string.sometext);
	 
	// Setting declared variables
	binding.set<VariableName>(variable);

## 自动更新布局

如果使用数据绑定，在数据发生变化时，库代码可以控制布局自动更新。然而，库仍然需要获得关于数据变化的通知。如果绑定的变量实现了 Observable 接口(不要跟 RxJava 的 Observable混淆了)就能解决这个问题。
对于像 int 和 boolean 这样的简单数据类型，库已经提供了合适的实现 Observable 的类型，比如 ObservableBoolean。还有一个 ObservableField 类型用于其它对象，比如字符串。


在更复杂的情况下，比如视图模型，有一个 BaseObservable 类提供了工具方法在变化时通知布局。就像上面在 setModel() 方法中看到那样，我们可以在模型变化之后通过调用 notifyChange() 来更新整个布局。

再看看 setAmount()，你会看到模型中只有一个属性发生了变化。这种情况下，我们不希望更新整个布局，只更新用到了这个属性的部分。为达此目的，可以在属性对应的 getter 上添加 @Bindable 注解。然后 BR 类中会产生一个字段，用于传递给 notifyPropertyChanged() 方法。这样，绑定库可以只更新确实依赖变化属性的部分布局。


	public class MyViewModel extends BaseObservable {
	   private Model model = new Model();
	 
	   public void setModel(Model model) {
	       this.model = model;
	       notifyChange();
	   }
	   
	   public void setAmount(int amount) {
	       model.setAmount(amount);
	       notifyPropertyChanged(BR.amount);
	   }
	 
	   @Bindable public String getText() { return model.getText(); }
	   @Bindable public String getAmount() { return Integer.toString(model.getAmount()); }
	}


# 局限性
在 MVVM 架构模式中，模型和视图模型主要通过数据绑定来进行互动。理想情况下，视图和视图模型不必相互了解。绑定应该是视图和视图模型之间的胶水，并且处理两个方向的大多数东西。然而，在Anroid中它们不能真实的分离：

* 你要保存和恢复状态，但现在状态在视图模型中。
* 你需要让视图模型知道生命周期事件（包含V层对VM层持有的引用，和LiveData对数据的持有）。
* 你可能会遇到需要直接调用视图方法的情况。



# 参考文章：

[MVVM 架构与数据绑定库](https://blog.csdn.net/hj7jay/article/details/54089656)  
[Android MVVM](https://blog.csdn.net/zzyawei/article/details/79590453)

