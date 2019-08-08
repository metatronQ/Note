# 内置注解

## @Override：用于覆写的方法，确保覆写的方法名的正确（大小写不同造成方法不同等）

## @Deprecated:用于不建议使用的方法、类（注释过的方法会出现删除线效果）  

## @SuppressWarnings():压制警告信息，即原本该警告的地方注释后不警告，可向括号中添加特定字符串数组的形式压制更多警告信息
* unchecked：执行了未检查的转换时的警告
* deprecation：使用了不赞成使用的类或方法的警告
* fallthrough：case未加入break的警告
* path：设置一个错误的类路径/源文件路径出现的警告
* serial：可序列化的类上缺少serialVersionUID定义的警告
* finally：任何finally子句不能正常完成时的警告
* all：以上所有情况的警告


# 自定义注解

自定义格式：
		
	public @interface 名称{
		数据类型 变量名称();
	}

可以向注解中设置内容（也可以设置数组）：通过指定属性名称接收指定内容  

默认值：定义了属性，就必须设置其内容，通过在定义注解属性时指定其默认值，可以在使用时不传参数
	
	public String key() default "内容";

使用枚举限制设置的内容：

	//枚举
	public enum MyName{
		MLDN,LXH,LI;
	}
	//定义注解属性时使用枚举类型
	public @interface MyDefaultAnnotation{
		public MyName name() default MyName.LXH;
	}

	//之后使用时只能传入枚举参数

## Retention:对自定义注解使用，指定注解的保存范围

value = Retention.

* SOURCE:只会保留到*.java中
* CLASS：*.java + *.class
* RUNTIME:*.java + *.class + JVM

# 通过反射取得注解

	//通过Class.forName("包.类");得到class实例

	//通过getMethod("方法名")得到方法

	//由Annotation an[] = method.getAnnotations();得到全部的注解

## 取得指定注解中的内容：通过isAnnotationPresent(注解.class)判断是否存在此注解，通过method.getAnnotation(注解.class)得到此注解的实例，通过.属性()获取内容


# @Target(value = ？):指定注解的使用位置

ANNOTATION_TYPE：只能用在注释的声明上
CONSTRUCTOR:构造方法的声明
METHOD：方法的声明
TYPE:类、接口、枚举类型
....

# @Docmented 


# @Inherited:用于注释定义上
标志此注释在父类被继承时继承入子类





