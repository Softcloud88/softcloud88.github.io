---
layout: post
title:  "属性动画"
date:   2016-06-19 23:27:54
categories: ui
---
## ViewPropertyAnimator：

ViewPropertyAnimator是最便利的绘制动画方法，对ViewPropertyAnimator的所有调用都会汇集到一个动画中。如：

	viewToAnimate.animate().alpha(0f).translationX(1000f);

---
## ObjectAnimator：

ObjectAnimator使用起来更加灵活，下面是个旋转图片的动画：

	mImage = (ImageView)findViewById(R.id.my_image);
	mRotation = ObjectAnimator.ofFloat(mImage, "rotationY", 0f, 360f);
	mRotation.setDuration(500);
	mRotation.addUpdateListener(new AnimatorUpdateListener()){
		@Override
		public void onAnimationUpdate(ValueAnimator animation){
			if(animation.getAnimatedFraction() >= 0.25f && isHead){
				mImage.setImageBitmap(mTailImage);
				isHead = false;
			}
			if (animation.getAnimatedFraction() >= 0.75 && !isHead){
				mImage.setImageBitmap(mHeadImage);
				isHead = true;
			}
		}
	};

ObjectAnimator也可以巧妙地实现一组动画同时播放，比如：

	ObjectAnimator anim = ObjectAnimator.ofFloat(view, "随便写", 1.0f, 0.0f).setDuration(1000);
	anim.start();
	anim.addUpdateListener(new AnimatorUpdateListener(){
		@Override
		public void onAnimationUpdate(ValueAnimator animation){
			float val = (float)animation.getAnimatedValue();
			view.setAlpha(val);
			view.setScaleX(val);
			view.setScaleY(val);
		}
	});
	
想在一个动画中更改多个效果也可以使用PropertyValuesHolder:

	PropertyValuesHolder pvhX = PropertyValuesHolder.ofFloat("alpha", 1f, 0f, 1f);
	PropertyValuesHolder pvhY = PropertyValuesHolder.ofFloat("scaleX", 1f, 0f, 1f);
	PropertyValuesHolder pvhZ = PropertyValuesHolder.ofFloat("scaleY", 1f, 0f, 1f);
	ObjectAnimator.ofPropertyValuesHolder(view, pvhX, pvhY,pvhZ).setDuration(1000).start();

---

## ValueAnimator：

ValueAnimator比ObjectAnimator更加灵活，可以自行定制evaluator，先看一个简单用法：

	ValueAnimator animator = ValueAnimator.ofFloat(0, 200f);
	animator.setTarget(view);
	animator.setDuration(500).start();
	animator.setInterpolator(new LinearInterpolator());
	animator.addUpdateListener(new AnimatorUpdateListner(){
		@Override
		public void onAnimationUpdate(ValueANimator animation){
			view.setTranslationY((Float)animation.getAnimationValue());
			view.setTranslationX(....);
			....
		}
	});
	
上面例子中的evaluator是Float，下面看一个自定义evaluator的例子

	ValueAnimator valueAnimator = new ValueANimator();
	valueAnimator.setDuration(2000);
	valueAnimator.setObject(new PointF(0, 0));
	valueAnimator.setInterpolator(new LinearInterpolator());
	valueAnimator.setEvaluator(new TypeEvaluator<PointF>(){
		@Override
		public PointF evaluate(float fraction, PointF startValue, PointF endValue){
			PointF point = new PointF();
			point.x = fraction * 200;
			point.y = point.x * 2;
			return point;
		}
	});
	valueAnimator.start();
	valueAnimator.addUpdateListener(new AnimatorUpdateListener(){
		@Override
		public void onAnimationUpdate(ValueAnimator animation){
			PointF point = (PointF)animation.getAnimatedValue();
			view.setX(point.x);
			view.setY(point.y);
		}
	});
		
## 监听器:
有时候，需要在动画结束时候删除某个元素，比如淡出后想把它干掉别占地方，可以添加监听器：

	ObjectAnimator anim = ObjectAnimator.ofFloat(mBlueBall, "alpha", 0.5f);  
          
    anim.addListener(new AnimatorListener(){  
  
        @Override  
        public void onAnimationStart(Animator animation) {  
            Log.e(TAG, "onAnimationStart");  
        }  
  
        @Override  
        public void onAnimationRepeat(Animator animation) {  
            Log.e(TAG, "onAnimationRepeat");  
        }  
  
        @Override  
        public void onAnimationEnd(Animator animation){  
        	Log.e(TAG, "onAnimationEnd");   
        }  
  
        @Override  
        public void onAnimationCancel(Animator animation){  
            Log.e(TAG, "onAnimationCancel");  
        }  
    });  
    anim.start();
    
除了new一个AnimatorListener外，还可以new一个AnimatorListenerAdaper()，她是实现了空方法的AnimatorListener。

	anim.addListener(new AnimatorListnerAdapter(){
		@Override
		public void onAnimationEnd(Animator animation){
			// todo
		}
	});

animator还有cancel()和end()方法：cancel动画立即停止，停在当前的位置；end动画直接到最终状态。

## AnimationSet：

AnimationSet可以指定动画播放的顺序：

	AnimatorSet animSet = new AnimationSet();
	animSet.play(anim1).with(anim2);
	animSet.play(anim2).with(anim3);
	animSet.play(anim4).after(anim3);
	animSet.setDuration(2000);
	animSet.start;

也可以在xml中定义动画集在代码中加载成AnimatorSet，下面是一个向上抛硬币的动画:

	res/animator/flip.xml
	
	<set xmlns:......
		android:ordering="together">
		<objectAnimator
			android:propertyName="rotationX"
			android:duration="400"
			android:valueFrom="0"
			android:valueTo="360"
			android:valueType="floatType"
			android:repeatMode="restart"
			android:repeatCount="3"
			android:interpolator="@android:interpolator/linear"/>
		<objectAnimator
			android:propertyName=:"translationY"
			android:duration="800"
			android:valueTo="-200"
			android:valueType="floatType"
			android:repeatMode="reverse"
			android:repeatCount="1"/>
	</set>
	
	
	AnimatorSet animSet = (AnimatorSet)AnimatorInflater.loadAnimator(this, R.animator.flip);
	animSet.setTarget(mImage);
	ObjectAnimator rotation = (ObjectAnimator)animSet.getChildAnimations().get(0);
	rotation.addUpdateListener(...//翻转换图片逻辑...);

	
