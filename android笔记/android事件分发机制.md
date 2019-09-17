# 首先必须了解的三大核心方法

## dispatchTouchEvent()

* **作用：分发传递点击事件，当当前View接收到事件时，该方法就会被调用**
* **存在于：Activity、ViewGroup、View**

## onInterceptTouchEvent()

* **作用：判断是否拦截某个事件，在ViewGroup的dispatchTouchEvent()内部调用**
* **只存在于：ViewGroup**

## onTouchEvent()

* **作用：处理点击事件，在dispatchTouchEvent()内部调用**
* **存在于：Activity、ViewGroup、View**


## 结构图：

![三方法关系的结构图](https://github.com/metatronQ/Note/blob/master/Images/%E6%96%B9%E6%B3%95%E4%BD%93.png)

## 关系伪代码：
	
	public boolean dispatchTouchEvent(MotionEvent ev){
		boolean consume = false;
		if(onInterceptTouchEvent(ev)){
			consume = onTouchEvent(ev);
		}else{
			consume = child.dispatchTouchEvent(ev);
		}
		return consume;
	}

# 事件分发的对象：Touch事件（MotionEvent）

**定义：当用户触摸屏幕时（View 或 ViewGroup派生的控件），将产生点击事件（Touch事件）**

## 注册监听touch事件

* **同注册点击事件的监听一样**：  
		
		findViewById(R.id.touch_test).setOnTouchListener(new View.OnTouchListener() {
	            @Override
	            public boolean onTouch(View view, MotionEvent motionEvent) {
	                switch (motionEvent.getAction()){
	                    case ACTION_DOWN:
	                        Toast.makeText(getApplication(),"down",Toast.LENGTH_SHORT).show();
	                }
	                return true;
	            }
	        });

* **onTouch与onClick的区别**:  
	1.onTouch优先于onClick执行  
	2.onTouch返回true时事件就被消费掉了，不会传递事件到onClick，返回false可以  
	3.onClick方法在onTouchEvent方法中被调用；
* **事件消费的含义**：传递到当前由于对事件进行处理（例：返回true），而不再向下传递；

## MotionEvent

[MotionEvent 详解](https://www.jianshu.com/p/0c863bbde8eb)

封装事件（时间动作等）的对象

## 事件列中常用事件类型：
* **MotionEvent.ACTION_DOWN**:按下View（开始）
* **MotionEvent.ACTION_UP**：抬起View（事件列的结束）
* **MotionEvent.ACTION_MOVE**：滑动
* **MotionEvent.ACTION_CANCEL**：结束事件（非人为原因，发生于ViewGroup向View传递事件的一种情况）

# 传递事件的三个对象及顺序

![传递顺序](https://github.com/metatronQ/Note/blob/master/Images/%E4%BA%8B%E4%BB%B6%E4%BC%A0%E9%80%92.png)

# 各版块的事件分发机制(源码)

## Activity

* **dispatchTouchEvent:**   

		public boolean dispatchTouchEvent(MotionEvent ev) {
	        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
	            onUserInteraction();
	        }

			//事件交给Activity所属的Window进行分发，如果返回true，，整个事件循环就结束，返回false则意味着事件没人处理，即所有View的onTouchEvent都返回了false，那么Activity的onTouchEvent方法就会被调用
	        if (getWindow().superDispatchTouchEvent(ev)) {
	            return true;
	        }
	        return onTouchEvent(ev);
	    }
		
		//找到superDispatchTouchEvent抽象方法的实现类：PhoneWindow，可以看到PhoneWindow直接将事件传递给了DecorView：一般为Activity的顶级View/根View，作为ViewGroup，通过setContentView所设置的View为它的一个子View
		public boolean superDispatchTouchEvent(MotionEvent event) {
		        return mDecor.superDispatchTouchEvent(event);
		    }
		



## ViewGroup:
定义： ViewGroup就是一组View的集合，它包含很多的子View和子VewGroup，是Android中所有布局的父类或间接父类，像LinearLayout、RelativeLayout等都是继承自ViewGroup的。但ViewGroup实际上也是一个View，只不过比起View，它多了可以包含子View和定义布局参数的功能


* **onInterceptTouchEvent**  
	

		//旧版
		//返回值为true，则开启拦截，返回值为false，则不拦截
		public boolean onInterceptTouchEvent(MotionEvent ev) {
		    return false;
		}

		//目前
		public boolean onInterceptTouchEvent(MotionEvent ev) {
	        if (ev.isFromSource(InputDevice.SOURCE_MOUSE)
	                && ev.getAction() == MotionEvent.ACTION_DOWN
	                && ev.isButtonPressed(MotionEvent.BUTTON_PRIMARY)
	                && isOnScrollbarThumb(ev.getX(), ev.getY())) {
	            return true;
	        }
	        	return false;
    	}

* **dispatchTouchEvent:**  
	
		public boolean dispatchTouchEvent(MotionEvent ev) {
        

			//如下：ViewGroup在两种情况下会判断是否要拦截当前事件：事件类型为ACTION_DOWN或者mFirstTouchTarget！=null
			//mFirstTouchTarget：当事件由ViewGroup的子元素成功处理时，该变量会被赋值并指向子元素，即一旦事件被当前的ViewGroup拦截，那么该变量就为null，当DOWN后面的一系列事件来临时，就不再调用onInterceptTouchEvent拦截了，后续事件即交由ViewGroup处理


			//FLAG_DISALLOW_INTERCEPT标志位：通过requestDIsallowInterceptTouchEvent方法设置，用于子View，一旦设置，将ViewGroup将无法拦截除了DOWN以外的其他点击事件，可以用于解决滑动冲突
				
            // Check for interception.
            final boolean intercepted;
            if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {
                final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                if (!disallowIntercept) {
                    intercepted = onInterceptTouchEvent(ev);
                    ev.setAction(action);
                } else {
                    intercepted = false;
                }
            } else {
                intercepted = true;
            }


			//遍历ViewGroup的所有子元素，然后判断子元素是否能够接受到点击事件：子元素是否在播放动画和点击事件的坐标是否落在子元素的区域内
			final View[] children = mChildren;
                        for (int i = childrenCount - 1; i >= 0; i--) {
                            final int childIndex = getAndVerifyPreorderedIndex(
                                    childrenCount, i, customOrder);
                            final View child = getAndVerifyPreorderedView(
                                    preorderedList, children, childIndex);

                            if (childWithAccessibilityFocus != null) {
                                if (childWithAccessibilityFocus != child) {
                                    continue;
                                }
                                childWithAccessibilityFocus = null;
                                i = childrenCount - 1;
                            }

                            if (!child.canReceivePointerEvents()
                                    || !isTransformedTouchPointInView(x, y, child, null)) {
                                ev.setTargetAccessibilityFocus(false);
                                continue;
                            }

                            newTouchTarget = getTouchTarget(child);
                            if (newTouchTarget != null) {
                                newTouchTarget.pointerIdBits |= idBitsToAssign;
                                break;
                            }

                            resetCancelNextUpFlag(child);



							//dispatchTransformedTouchEvent内部通过判断child是否为null来决定调用的是ViewGroup还是子View的dispatchTouchEvent方法
                            if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                                mLastTouchDownTime = ev.getDownTime();
                                if (preorderedList != null) {

                                    for (int j = 0; j < childrenCount; j++) {
                                        if (children[childIndex] == mChildren[j]) {
                                            mLastTouchDownIndex = j;
                                            break;
                                        }
                                    }
                                } else {
                                    mLastTouchDownIndex = childIndex;
                                }
                                mLastTouchDownX = ev.getX();
                                mLastTouchDownY = ev.getY();
                                newTouchTarget = addTouchTarget(child, idBitsToAssign);
                                alreadyDispatchedToNewTouchTarget = true;
                                break;
                            }


			//当子View符合条件时，会对mFirstTouchEvent进行赋值并终止遍历；
			//当遍历所有子View都没有被合适的处理：1.ViewGroup没有子元素；2.子View处理了点击事件，但是其dispatchTouchEvent返回了false，ViewGroup会自己处理点击事件

			// Dispatch to touch targets.
            if (mFirstTouchTarget == null) {
                // No touch targets so treat this as an ordinary view.
                handled = dispatchTransformedTouchEvent(ev, canceled, null,TouchTarget.ALL_POINTER_IDS);


          }



	1. Android事件分发是先传递到ViewGroup，再由ViewGroup传递到View的。

	2. 在ViewGroup中可以通过onInterceptTouchEvent方法对事件传递进行拦截，onInterceptTouchEvent方法返回true代表不允许事件继续向子View传递，返回false代表不对事件进行拦截，默认返回false。

	3. 子View中如果将传递的事件消费掉，ViewGroup中将无法接收到任何事件。



## View（旧）

* **dispatchTouchEvent:有个很重要的点：当dispatchTouchEvent在进行事件分发的时候，只有前一个action返回true，才会触发后一个action（onTouch的false是带有迷惑性的）**  
		
		public boolean dispatchTouchEvent(MotionEvent event) {
			//注意下面三个判断条件
		    if (mOnTouchListener != null && (mViewFlags & ENABLED_MASK) == ENABLED && mOnTouchListener.onTouch(this, event)) {
		        return true;
		    }
			
			//当onTouch返回false时，则dispatchTouchEvent的返回值与onTouchEvent保持一致
	    	return onTouchEvent(event);
		}

* 可以看到上方的三个判断，当三个条件都为真时，则返回true：

	* mOnTouchListener：   
	view中的setOnTouchListener方法为该变量赋值：  
		
			public void setOnTouchListener(OnTouchListener l) {
	    		mOnTouchListener = l;
			}
	**即**：只要给控件注册了点击事件，该变量就一定被赋值

	* mViewFlags & ENABLED_MASK) == ENABLED：   
	该条件是判断当前点击的控件是否是enable的，按钮是默认enable的，当然可以通过在控件中指定clickable属性来达到相同的效果

	* **mOnTouchListener.onTouch(this, event)**：  
	回调控件注册touch事件时的onTouch方法，即在在控件可点击的情况下，判断的结果取决于该方法返回的真假，当然可以在onTouch方法中手动指定真假

* **onTouchEvent()**  

		public boolean onTouchEvent(MotionEvent event) {
	    final int viewFlags = mViewFlags;
	    if ((viewFlags & ENABLED_MASK) == DISABLED) {
	        // A disabled view that is clickable still consumes the touch
	        // events, it just doesn't respond to them.
	        return (((viewFlags & CLICKABLE) == CLICKABLE ||
	                (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE));
	    }
	    if (mTouchDelegate != null) {
	        if (mTouchDelegate.onTouchEvent(event)) {
	            return true;
	        }
	    }

		
		//第一个点，当控件可点击时，会进入事件的switch判断
	    if (((viewFlags & CLICKABLE) == CLICKABLE ||
	            (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)) {
	        switch (event.getAction()) {
	            case MotionEvent.ACTION_UP:
	                boolean prepressed = (mPrivateFlags & PREPRESSED) != 0;
	                if ((mPrivateFlags & PRESSED) != 0 || prepressed) {
	                    // take focus if we don't have it already and we should in
	                    // touch mode.
	                    boolean focusTaken = false;
	                    if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
	                        focusTaken = requestFocus();
	                    }
	                    if (!mHasPerformedLongPress) {
	                        // This is a tap, so remove the longpress check
	                        removeLongPressCallback();
	                        // Only perform take click actions if we were in the pressed state
	                        if (!focusTaken) {
	                            // Use a Runnable and post this rather than calling
	                            // performClick directly. This lets other visual state
	                            // of the view update before click actions start.
	                            if (mPerformClick == null) {
	                                mPerformClick = new PerformClick();
	                            }
	                            if (!post(mPerformClick)) {


									//第二个点，经过一系列判断之后会执行这个方法
	                                performClick();
	                            }
	                        }
	                    }
	                    if (mUnsetPressedState == null) {
	                        mUnsetPressedState = new UnsetPressedState();
	                    }
	                    if (prepressed) {
	                        mPrivateFlags |= PRESSED;
	                        refreshDrawableState();
	                        postDelayed(mUnsetPressedState,
	                                ViewConfiguration.getPressedStateDuration());
	                    } else if (!post(mUnsetPressedState)) {
	                        // If the post failed, unpress right now
	                        mUnsetPressedState.run();
	                    }
	                    removeTapCallback();
	                }
	                break;
	            case MotionEvent.ACTION_DOWN:
	                if (mPendingCheckForTap == null) {
	                    mPendingCheckForTap = new CheckForTap();
	                }
	                mPrivateFlags |= PREPRESSED;
	                mHasPerformedLongPress = false;
	                postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
	                break;
	            case MotionEvent.ACTION_CANCEL:
	                mPrivateFlags &= ~PRESSED;
	                refreshDrawableState();
	                removeTapCallback();
	                break;
	            case MotionEvent.ACTION_MOVE:
	                final int x = (int) event.getX();
	                final int y = (int) event.getY();
	                // Be lenient about moving outside of buttons
	                int slop = mTouchSlop;
	                if ((x < 0 - slop) || (x >= getWidth() + slop) ||
	                        (y < 0 - slop) || (y >= getHeight() + slop)) {
	                    // Outside button
	                    removeTapCallback();
	                    if ((mPrivateFlags & PRESSED) != 0) {
	                        // Remove any future long press/tap checks
	                        removeLongPressCallback();
	                        // Need to switch from pressed to not pressed
	                        mPrivateFlags &= ~PRESSED;
	                        refreshDrawableState();
	                    }
	                }
	                break;
	        }

			//第三个点，当控件可点击时，返回true，使后续事件接着传递
	        return true;
	    }
		//第四个点，当控件不可点击时，返回false，后续事件不能传递
	    return false;
		}


		//此方法可以看出，当mOnClickListener不为null时，就会调用其onClick方法
		public boolean performClick() {
		    sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
		    if (mOnClickListener != null) {
		        playSoundEffect(SoundEffectConstants.CLICK);
		        mOnClickListener.onClick(this);
		        return true;
		    }
		    return false;
		}


		//而mOnClickListener的赋值是由下面方法完成的，即当为控件注册点击事件时，都会为该变量赋值
		public void setOnClickListener(OnClickListener l) {
		    if (!isClickable()) {
		        setClickable(true);
		    }
		    mOnClickListener = l;
		}

# 事件分发的流程图

* **是一个由上往下且由下往上的过程**

# 解决实际的问题

## 1.EditText 点击空白区域有效：先弹出键盘，再获得EditText的焦点

## 2.EditText 点击空白区域隐藏软键盘：隐藏软键盘，取消EditText的焦点
	
[点击空白区域隐藏软键盘](https://blog.csdn.net/zhuod/article/details/52489504)


## 3.滑动冲突


# 参考：

* **书籍：Android开发艺术探索**
* **博客：**  
	**[Android事件分发机制完全解析，带你从源码的角度彻底理解](https://blog.csdn.net/guolin_blog/article/details/9097463)**  
	**[Android事件分发机制详解：史上最全面、最易懂](https://www.jianshu.com/p/38015afcdb58)**