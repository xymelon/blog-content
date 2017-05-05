Title: Android仿腾讯新闻创意截屏
Date: 2015-10-27 19:34
Category: Technology
Tags: Android
Slug: android-novcapture
Author: xymelon

前不久使用腾讯新闻的时候，偶然间发现一个很酷炫的功能——创意截屏，大概就是截取当前屏幕生成一张图片，然后可以在这张图片上添加多个自定义标签。功能支持多种手势操作，包括单手拖动、缩放、旋转、删除；双手缩放、旋转等，并可动态更改文字标签内容，使用过程中基本没有卡顿现象发生。本文主要就讲讲我是怎么实现创意截屏功能，当然条条大路通罗马，每个人可能都有自己的实现方式，欢迎大家拍砖并提出建议。

<!-- PELICAN_END_SUMMARY -->

### 截屏预览 ###

首先看看实现后的整个创意截屏的样子吧，最直观。PS：在我的几个测试机上基本没有卡顿现象，很流畅。

![image](/images/novcapture.gif)

（图1）打开APP后，截取屏幕进入添加标签即可体验。

### 关键细节 ###

起初看到创意截屏这个功能，私以为比较简单，结果在进行编码过程中，才发现里面有很多细节需要进行考虑。

1. 标签（图中虚线框容器）不能完全使用ViewGroup进行拼装，不然进行缩放、旋转、拖动时会出现两个问题：
	- 性能问题，会出现明显的卡顿现象;
	- 显示问题，当标签进行缩放时，其实标签左上删除、右下缩放与旋转按钮一直保持大小不变。若使用整体缩放，则会出现一起放大缩小，不符合效果；当然也可进行特殊处理，比如针对这俩按钮进行反比例缩放。
	
	在本文中，**关键解决方案在于将整个标签内容ViewGroup生成Bitmap，然后维护标签、删除、拖放按钮等的绘制矩阵，在onDraw中自定义绘画**，非常灵活，性能很不错。
	
1. 缩放、旋转、拖动相关，其中每个功能都有多种实现方式，比如旋转可用库函数setRotation或canvas.rotate画布操作、拖动可动态改变margin或使用canvas.translate实现等。先不说使用库函数会遇到兼容低版本问题，而且对性能也有很大影响，如当缩放比例很大时，拖动过程中会出现非常明显的卡顿现象。在本文中，**针对标签的使用场景（频繁缩放、旋转、拖动等）选用画布Canvas操作才是上策**。
	
1. 事件分发，当容器中存在多个标签时，就会判断是哪个标签接收并处理事件。通过上段知道，我放弃使用库函数进行缩放、旋转、拖动，选用画布Canvas进行操作，同样，Canvas操作也带来了另外的问题，如：
	* 事件接收的坐标点不会跟着Canvas改变，如View在A坐标canvas.translate移动到B坐标后，View接收事件的坐标点还是会停留在原A坐标。因此，我们还需**手动维护虚线框、删除、缩放与旋转按钮的当前绘制矩阵，以便事件分发时判定**。
	* 当View旋转后，如何判定坐标是否在View区域内。如下面图2所示，假设蓝色矩形为我们维护的原始矩阵matrix，当我们调用matrix.postRotate进行旋转操作后，matrix会变成图2中的虚线框区域，而实际上应该为绿色区域，若不进行处理，则会判断错误。**本文中使用了小技巧，即我们维护一个未旋转的matrix(包含缩放、拖动)和一个旋转角度变量rotation。当判断图中点A是否在绿色区域内，则可将其反向旋转rotation后变成点B，判断点B是否在蓝色矩形中即可**。
	
		![image](/images/novcapture_1.png)

		（图2）坐标点判断示例。

	
1. 虚线框绘画，使用<a href="http://developer.android.com/reference/android/graphics/DashPathEffect.html" target="_blank">DashPathEffect</a>绘制虚线框时，为了能让其动起来，不可避免地会根据phase创建很多对象，若不能有效处理，则会导致频繁地内存回收，影响性能。本文中优化方式是根据虚线框实、虚线长度创建多个对象循环使用，如：

		private PathEffect getPathEffect() {
			// phase值在 0 - mDashMinSize 之间循环
        	mDashPhase %= mDashMinSize;
        	PathEffect effect = mPathEffects[mDashPhase];
        	// 判定当前phase的DashPathEffect对象是否创建，是则循环使用，反之新建
	        if (effect == null) {
    	        effect = new DashPathEffect(mDashFloats, mDashPhase);
        	    mPathEffects[mDashPhase] = effect;
        	}
	        mDashPhase++;
    	    return effect;
	    }

1. 软键盘遮挡输入框或界面上移。当我们需要编辑文本的时候，基本都会遇到软键盘把输入框遮挡或导致界面整个上提的问题，google一圈下来基本都是说调整android:windowSoftInputMode属性为"adjustResize"或"adjustPan"，而这并不能完全解决问题还需要调整布局，比较麻烦。本文使用的方法是将输入框放入dialog或popupwindow中，这不但可以解决前述问题，还能将代码组织得更模块化。

### 代码分析 ###

![image](/images/novcapture_2.png)

（图2）代码组织结构。

先说说整个代码结构吧，核心在图2 labeller 库中的`CaptureView`和`DashFrameLayout`。

#### CaptureView ####

本类主要负责整个创意截屏界面处理，包含内容、下方添加标签、输入框等区域。
有个关键地方就是当设置完成图片内容后，内容区域的高宽度需要完全匹配图片大小，否则当进行拖动操作时，标签会移出图片，最后的保存操作时也会出现空白区域，非常不美观。关键代码如下：

		private void setImageBitmap(Bitmap bitmap) {
        	mContainerLayout.getLayoutParams().width = bitmap.getWidth();
	        mContainerLayout.getLayoutParams().height = bitmap.getHeight();
    	    mImageIV.getLayoutParams().width = bitmap.getWidth();
        	mImageIV.getLayoutParams().height = bitmap.getHeight();
	        mImageIV.setImageBitmap(bitmap);
    	}

添加标签时，还需维护当前处于焦点的标签，以便处理是否显示虚线框，如下：

		private void addLabel(View view) {
        	final DashFrameLayout dashFrameLayout = new DashFrameLayout(getContext());
        	// 添加监听器，记录状态
	        dashFrameLayout.setLabelListener(new DashFrameLayout.LabelListener() {
    	        @Override
        	    public void onFocus() {
            	    mFocusLabel = dashFrameLayout;
            	}

	            @Override
    	        public void onClick() {
        	        onLabelClick();
            	}
	        });
    	    dashFrameLayout.addView(view);
	
    	    hideFocusLabelDashBorder();
        	mFocusLabel = dashFrameLayout;

	        mContainerLayout.addView(dashFrameLayout);
	    }

#### DashFrameLayout ####

本类应该是创意截屏功能的核心所在，即标签（虚线框容器），包含几大部分：左上角删除按钮、右下角拖放与旋转按钮、中间虚线框容器、标签内容。

初始时，需计算各个部分的绘制矩阵，并将标签内容生成bitmap，提高性能。核心代码如下：

	        Rect rect = new Rect();
            mChildView.getLocalVisibleRect(rect);

            // 计算虚线框绘制位置，注意虚线框与label之间存在margin
            mDashRectF.left = rect.left - margin;
	        mDashRectF.top = rect.top - margin;
		    mDashRectF.right = rect.right + margin;
	        mDashRectF.bottom = rect.bottom + margin;

            // 计算删除按钮初始中心坐标
            mDelPoint[0] = mDashRectF.left;
            mDelPoint[1] = mDashRectF.top;

            // 计算拖放按钮初始中心坐标
            mDragPoint[0] = mDashRectF.right;
            mDragPoint[1] = mDashRectF.bottom;

            // 将view转换为bitmap，便于旋转、缩放时提高性能
            mLabelBitmap = ImageUtils.captureScreenByDraw(mChildView);

然后根据维护的矩阵，自定义onDraw方法，如下：

		@Override
    	protected void onDraw(Canvas canvas) {
        	// 画标签
        	canvas.drawBitmap(mLabelBitmap, mViewMatrix, null);
        	if (mShowDashBorder) {
         	    // 画虚线框
            	mDashPaint.setPathEffect(getPathEffect());
	            canvas.save();
    	        canvas.concat(mViewMatrix);
        	    canvas.drawPath(mDashPath, mDashPaint);
            	canvas.restore();

            	// 画删除按钮：左上角
	            canvas.drawBitmap(mDelBitmap, null, mDelDrawRectF, null);

    	        // 画拖放按钮：右下角
        	    canvas.drawBitmap(mDragBitmap, null, mDragDrawRectF, null);
        	}
    	}

其他很关键的比如事件分发，因代码过长，不在此累述，请参考源代码中onTouchEvent实现方法。PS：单手旋转、缩放操作，可将其认为双手操作，只不过其中一只手固定在标签中心。

有疑问或者觉得不对的地方还请指正，谢谢。

创意截屏源码：<a href="https://github.com/xymelon/nov-capture" target="_blank">https://github.com/xymelon/nov-capture</a>
