# MVP的简单实现和简单原理
## 接口管理：Contract（可有可无，方便对大量的接口进行管理）
	public interface Contract{
	    interface View{
	        //V层展示数据
	        void showData(String string);
	    }
	
	    interface Model{
	        //M层处理并返回数据
	        String doData();
	    }
	
	    interface Presenter{
	        //P层得到M层的数据并传给V层
	        void getData();
	    }
	}


## M层（model）

* 重写方法处理数据并返回数据

		public class ModelTest implements Contract.Model {
		
		    static ModelTest modelTest;
		    public static ModelTest getInstance(){
		        if (modelTest == null){
		            modelTest = new ModelTest();
		        }
		
		        return modelTest;
		    }
		
		    @Override
		    public String doData() {
		        return "处理好的数据";
		    }
		}

* 需要创建一个返回Medel层实例的方法：此方法比较重要，必须要获取到M层子类的实例才能调用到重写过的方法，而且有了这个方法，无论在哪个层，只需要ModelTest.getInstance()就可以获得实例并取得数据处理的结果；

## V层（View层）

* 显示数据：重写展示数据的具体逻辑

		 public void showData(String string) {
		        showData.setText(string);
		    }

* 实例化P层对象同时传入M层子类实例和V层子类实例，这样便可得到重写的子类方法，整个逻辑变贯通了

		PresenterTest presenterTest = new PresenterTest(ModelTest.getInstance(),this);
        presenterTest.getData();


## P层（Presenter层）

* mvp架构的中间人，通过p层连接M层和V层，M层取得数据之后，传给P层，P层再交给View层
* 只负责调用方法，而不负责展示或处理数据的逻辑
* 通过构造方法取得View接口和Model接口的实例，然后重写方法调用view接口的方法并调用Model接口的方法传入数据展示
		
		public class PresenterTest implements Contract.Presenter {
		
		    private Contract.Model model;
		    private Contract.View view;
		
		    public PresenterTest(Contract.Model model, Contract.View view){
		        this.model = model;
		        this.view = view;
		    }
		    @Override
		    public void getData() {
		        view.showData(model.doData());
		    }
		}

# Base层：
**Base层作为基本层，实现一些其他版块的共同的方法，由其他版块继承，包括BaseActivity，BasePresenter，BaseModel，相当于Base层为实现了简易MVP架构的层，其他层继承这个层并执行具体的业务，此层一般多为抽象类和接口**

* BaseActivity：注入了继承BasePresenter的泛型，用以跟P层通信，如返回结果给P层


# 易扩展性：如果处理数据的方法有多个方法或展示数据的方法有多个方法：

* 在接口中多加几个M层和对应多加几个P层，V层不变增加逻辑：一V多P多M

* 在M、P、V层多加几个方法：1M1V1P

* 在M、V层多加几个方法，在P层链接方法中增加链接方法的逻辑：1M1V1P,P不变

* 多加几个M层，V层多添加几个方法，P层不变：1V1P多M
* 。。。。  


### 自己的对整个过程的理解：P层就是一个中转站，M层则只负责运输数据，V层则只负责展示数据，当M层将数据传入P层时，P层将数据打包装给V层，由V层运去展示；

### 注：其实任意层都可以添加对数据的处理，甚至也可以添加对数据的展示，但是这样做会使MVP各自的职责不清不楚，本来MVP就是一种牺牲代码量简化逻辑的一种分工化的思想，越俎代庖最好越少越好。