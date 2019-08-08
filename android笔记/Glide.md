[文档](https://muyangmin.github.io/glide-docs-cn/)
## 基本用法  
* Glide.with().load(url).into(imageView)


## 占位符
* Glide允许用户指定三种不同的占位符  
注： Transformation仅被应用于被请求的资源，而不会对任何占位符使用  

* palceholder(占位符)：
	* 占位符是当请求正在执行时被展示的 Drawable。当请求成功完成时，占位符会被请求到的资源替换。如果被请求的资源是从内存中加载出来的，那么占位符可能根本不会被显示。如果请求失败并且没有设置 error Drawable ，则占位符将被持续展示。类似地，如果请求的url/model为 null ，并且 error Drawable 和 fallback 都没有设置，那么占位符也会继续显示。
* error(错误符)：
	* error Drawable 在请求永久性失败时展示。error Drawable 同样也在请求的url/model为 null ，且并没有设置 fallback Drawable 时展示。
* fallback(后备回调符):
	* fallback Drawable 在请求的url/model为 null 时展示。设计 fallback Drawable 的主要目的是允许用户指示 null 是否为可接受的正常情况。例如，一个 null 的个人资料 url 可能暗示这个用户没有设置头像，因此应该使用默认头像。然而，null 也可能表明这个元数据根本就是不合法的，或者取不到。 默认情况下Glide将 null 作为错误处理，所以可以接受 null 的应用应当显式地设置一个 fallback Drawable 。


			Glide.with(fragment)
			 	.load(url)  
				.placeholder(R.drawable.palce)
			 	.error(R.drawable.error)
				.fallback(R.drawable.fall)
			 	.into(view);

## 选项

### 请求选项

Glide中的大部分设置项都可以直接应用在 Glide.with() 返回的 RequestBuilder 对象上。（Gilde.with() -> RequestBuilder）

可用的选项包括（但不限于）：

* 占位符(Placeholders)
* 转换(Transformations)
* 缓存策略(Caching Strategies)
* 组件特有的设置项，例如编码质量，或Bitmap的解码配置等。

### RequestOptions
* 共享请求：apply实现共享  

		RequestOptions sharedOptions = new RequestOption()
							.placeholder(palceholder)
							.fitCenter()
		Gilde.with(fragment).load(url).apply(sharedOptions).into(imageView1);
		...
		//如果 RequestOptions 对象之间存在相互冲突的设置，那么只有最后一个被应用的 RequestOptions 会生效。

### 过渡选项(TransitionOption)

TransitionOptions 用于决定你的加载完成时会发生什么.

[查看其三个直接子类获取过渡方法](https://muyangmin.github.io/glide-docs-cn/javadocs/400/com/bumptech/glide/TransitionOptions.html)

使用 TransitionOption 可以应用以下变换：

* View淡入
* 与占位符交叉淡入
* 或者什么都不发生

应用一个交叉淡入：
	
	import static com.bumptech.glide.load.resource.drawable.DrawableTransitionOptions.withCrossFade;

	Glide.with(fragment)
	    .load(url)
	    .transition(withCrossFade())
	    .into(view);
		
	//假如你请求加载一个 Bitmap ，你需要使用 BitmapTransitionOptions ，而不是 DrawableTransitionOptions

### RequestBuilder
RequestBuilder 是Glide中请求的骨架，负责携带请求的url和你的设置项来开始一个新的加载过程。 

使用 RequestBuilder 可以指定：

* 你想加载的资源类型(Bitmap, Drawable, 或其他)
* 你要加载的资源地址(url/model)
* 你想最终加载到的View
* 任何你想应用的（一个或多个)RequestOption 对象
* 任何你想应用的（一个或多个）TransitionOption 对象
* 任何你想加载的缩略图 thumbnail()

*  **构造RequestBuilder对象**  


		//通过先调用 Glide.with()然后再调用某一个 as 方法来完成
		RequestBuilder<Drawable> requestBuilder = Glide.with(fragment).asDrawable();
	
		//先调用 Glide.with() 然后 load()
		RequestBuilder<Drawable> requestBuilder = Glide.with(fragment).load(url);

* **选择资源类型**：  
默认情况下你会得到一个 Drawable RequestBuilder ，但你可以使用 as... 系列方法来改变请求类型。例如，如果你调用了 asBitmap() ，你就将获得一个 BitmapRequestBuilder 对象，而不是默认的 Drawable RequestBuilder。 

* **应用RequestOptions**  

		//可以使用 apply() 方法应用 RequestOptions ，使用 transition() 方法应用 TransitionOptions
		RequestBuilder<Drawable> requestBuilder = Glide.with(fragment).asDrawable();
		requestBuilder.apply(requestOptions);
		requestBuilder.transition(transitionOptions);

		//RequestBuilder 也可以被复用于开始多个请求：
		RequestBuilder<Drawable> requestBuilder =
        Glide.with(fragment)
            .asDrawable()
            .apply(requestOptions);

		for (int i = 0; i < numViews; i++) {
		   ImageView view = viewGroup.getChildAt(i);
		   String url = urls.get(i);
		   requestBuilder.load(url).into(view);
		}

* **缩略图请求** 
 
Glide 的 thumbnail()允许你指定一个 RequestBuilder 以与你的主请求并行启动。thumbnail() 会在主请求加载过程中展示。如果主请求在缩略图请求之前完成，则缩略图请求中的图像将不会被展示  

		Glide.with(fragment)
				.load(url)
				.thumbnail(Glide.with(fragment)
		  			.load(thumbnailUrl))
		  		.into(imageView);


仅仅想加载一个本地图像，或者你只有一个单独的远程 URL:	使用 Glide 的 override 或 sizeMultiplier API 来强制 Glide 在缩略图请求中加载一个低分辨率图像
	
	int thumbnailSize = ...;
	Glide.with(fragment)
	  .load(localUri)
	  .thumbnail(Glide.with(fragment)
	    .load(localUri)
	    .override(thumbnailSize))
	  .into(view);

thumbnail() 方法有一个简化版本，它只需要一个 sizeMultiplier 参数。如果你只是想为你的加载相同的图片，但尺寸为 View 或 Target 的某个百分比的话特别有用： 
 
	Glide.with(fragment)
	  .load(localUri)
	  .thumbnail(/*sizeMultiplier=*/ 0.25f)
	  .into(imageView);

* **在失败时开始新的请求**

从 Glide 4.3.0 开始，你现在可以使用 error API 来指定一个 RequestBuilder，以在主请求失败时开始一次新的加载。例如，在请求 primaryUrl 失败后加载 fallbackUrl：

	Glide.with(fragment)
	  .load(primaryUrl)
	  .error(Glide.with(fragment)
	      .load(fallbackUrl))
	  .into(imageView);

如果主请求成功完成，这个error RequestBuilder 将不会被启动。如果你同时指定了一个 thumbnail() 和一个 error() RequestBuilder，则这个后备的 RequestBuilder 将在主请求失败时启动，即使缩略图请求成功也是如此。

### 组件选项  

Option 类是给Glide的组件添加参数的通用办法，包括 ModelLoaders , ResourceDecoders , ResourceEncoders , Encoders 等等。一些Glide的内置组件提供了设置项，自定义的组件也可以添加设置项

	//Option 通过 RequestOptions 类应用到请求上：
	Glide.with(context)
	  .load(url)
	  .option(MyCustomModelLoader.TIMEOUT_MS, 1000L)
	  .into(imageView);

	//你也可以创建一个全新的 RequestOptions 对象:
	RequestOptions options = new RequestOptions().set(MyCustomModelLoader.TIMEOUT_MS, 1000L);
	
	Glide.with(context)
	  .load(url)
	  .apply(options)
	  .into(imageView);

## 变换

### 内置类型：
* CenterCrop
* FitCenter
* CircleCrop

### 应用

通过 RequestOptions 类可以应用变换：

* **默认变换:** 

		Glide.with(fragment)
		  .load(url)
		  .fitCenter()
		  .into(imageView);

		//或使用 RequestOptions
		RequestOptions options = new RequestOptions();
		options.centerCrop();
		
		Glide.with(fragment)
		    .load(url)
		    .apply(options)
		    .into(imageView);
* **多重变换:**


默认情况下，每个 transform() 调用，或任何特定转换方法(fitCenter(), centerCrop(), bitmapTransform() etc)的调用都会替换掉之前的变换。

如果你想在单次加载中应用多个变换，请使用 MultiTransformation类，或其快捷方法 .transforms():

* MultiTransformation类

		//注：向 MultiTransformation 的构造器传入变换参数的顺序，决定了这些变换的应用顺序。

		Glide.with(fragment)
		  .load(url)
		  .transform(new MultiTransformation(new FitCenter(), new YourCustomTransformation())
		  .into(imageView);

* 快捷方法:.transforms()

		Glide.with(fragment)
		  .load(url)
		  .transform(new FitCenter(), new YourCustomTransformation())
		  .into(imageView);

### 定制变换

[有点麻烦自查文档](https://muyangmin.github.io/glide-docs-cn/doc/transformations.html#bitmaptransformation)  

* 注意：对于任何Transformation子类，都有三个方法必须实现：  
	1.equals()  
	2.hashCode()  
	3.updateDiskCacheKey()  

## 特殊行为

### 重用变换

equals()
hashCode()
updateDiskCacheKey

### ImageView的自动变换

在Glide中，当你为一个 ImageView 开始加载时，Glide可能会自动应用 FitCenter 或 CenterCrop ，这取决于view的 ScaleType 。如果 scaleType 是 CENTER_CROP , Glide 将会自动应用 CenterCrop 变换。如果 scaleType 为 FIT_CENTER 或 CENTER_INSIDE ，Glide会自动使用 FitCenter 变换。

当然，你总有权利覆写默认的变换，只需要一个带有 Transformation 集合的 RequestOptions 即可。另外，你也可以通过使用 dontTransform() 确保不会自动应用任何变换。

### 自定义资源

因为 Glide 4.0 允许你指定你将解码的资源的父类型，你可能无法确切地知道将会应用何种变换。例如，当你使用 asDrawable() (或就是普通的 with() ，因为 asDrawable() 是默认情形)来加载 Drawable 资源时，你可能会得到 BitmapDrawable 子类，也有可能得到 GifDrawable 子类。

为了确保你添加到 RequestOptions 中的任何变换都会被使用，Glide将 Transformation 添加到一个Map中保存，其Key为你提供变换的资源类型。当资源被成功解码时，Glide使用这个Map来取回对应的 Transformation 。

Glide可以将 Bitmap Transformation应用到 BitmapDrawable , GifDrawable , 以及 Bitmap 资源上，因此通常你只需要编写和应用 Bitmap Transformation 。然而，如果你添加了额外的资源类型，你可能需要考虑派生 RequestOptions 类，并且，在内置的这些 Bitmap Transformations 之外，你还需要为你的自定义资源类型提供一个 Transformation 。


## 目标
[暂时看不懂](https://muyangmin.github.io/glide-docs-cn/doc/targets.html)

### 关于Target

在Glide中，Target 是介于请求和请求者之间的中介者的角色。Target 负责展示占位符，加载资源，并为每个请求决定合适的尺寸。被使用得最频繁的是 ImageViewTargets ，它用于在 ImageView 上展示占位符、Drawable 和 Bitmap 。用户还可以实现自己的 Target ，或者从任何可用的基类派生子类。

### 指定目标  

	Target<Drawable> target = 
	  Glide.with(fragment)
	    .load(url)
	    .into(new Target<Drawable>() {
	      ...
	    });

	//辅助方法 into(ImageView) ，它接受一个 ImageView 参数并为其请求的资源类型包装了一个合适的 ImageViewTarget：
	Target<Drawable> target = 
	  Glide.with(fragment)
	    .load(url)
	    .into(imageView);
 into(Target) 和 into(ImageView) 都返回了一个 Target 实例  

## 过渡

[m](https://muyangmin.github.io/glide-docs-cn/doc/transitions.html)

