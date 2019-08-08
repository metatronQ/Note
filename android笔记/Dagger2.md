# dagger2是什么

* ***请不要忘记Dagger2的实质是依赖注入，即当作为依赖注入来使用***

**用注解代替传统依赖注入，主要用于模块间解耦，提高代码的健壮性和可维护性**


# 依赖提供方：先Module再Inject

## @Module＆@Provides

* Module：用于标注提供依赖的类；由于第三方库的构造函数是不能加@Inject注解的，或者由于提供的构造函数是带参数的，不方便@Inject注解

* Provides：用于标注Module所标注的类中的方法，在该方法被依赖时被调用，从而把预先提供好的对象当做依赖给标注了@Inject的变量赋值

## @Inject构造方法

标记构造函数，Dagger2通过@Inject注解可以在需要这个类实例的时候找到这个构造函数并把相关实例构造处理，以此来为被@Inject标记了的变量提供依赖


# 依赖注入容器：@Component：必须有

@Component(module = 依赖来自的类)
接口方法中写返回值为void的方法传入被依赖的目标
@Component(dependencies = 其他的Component)
接口方法中写返回值不为void的不传入参数的方法

用于标注**接口**，时依赖需求方和依赖提供方之间的桥梁；被Component标注的接口在编译时会生成该接口的实现类（Dagger+Component名字），通过调用这个实现类的方法完成注入；

Component接口接口中主要定义一些提供依赖的声明：@Component(modules = {a.class,...},dependencies = {Component.class,...})


# 依赖需求方：@Inject Object
Dagger2遇到@Inject标记的成员属性，就会去查看该成员类的构造函数，如果构造函数也被@Inject标记,则会自动初始化，完成依赖注入

	DaggerAppComponent.builder()
	               .build()         
			       .inject(this);

# @Scope

自定义注解：限定注解的作用域，实现单例模式

* @Singleton：@Scope定义的注解，一般用于标记全局单例


# 简单实例：
[上手](https://www.jianshu.com/p/22c397354997)

# Dagger2 + MVP