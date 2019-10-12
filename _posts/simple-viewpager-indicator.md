---
title: android simple ViewPager Indicator
date: 2016-12-22 16:27:15
categories: Android

---
<Excerpt in index | 首页摘要> 
> android simple ViewPager Indicator
>
> Android一个简单的ViewPager指示器自己实现
> <!-- more -->
> <The rest of contents | 余下全文> 



先看看Demo图

![](http://www.gloomyer.com/img/img/simple-viewpager-indicator.gif)

效果图.如上。

####  原理  ####

简单说一下，这个实现很简单。

首先我们，来了解下ViewPager的OnPageChangeListener这个回调

它有三个方法:

```java
public void onPageScrollStateChanged(int state) {}
/*
 ViewPager状态发生变化触发的回调
 它有三个取值
 0: 什么都不做
 1: 开始滚动
 2: 惯性滚动(手指离开屏幕，惯性滚动会触发这个状态)
*/
public void onPageSelected(int position) {}
/*
 这个是ViewPager完全停止之后会触发的回调，position代表当前选中的坐标
*/
public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {}
/*
	这个 是重点，今天需要借助它来实现上面的效果
	他三个参数分别表示:
		position:已经移动的满足一页的坐标
		positionOffset:不满足的偏移量百分比形式
		positionOffsetPixels:不满足的偏移量的像素点形式
		
	怎么理解第一个的意思呢。
	比如：0__0.5__1__1.5__2
	类似这样，如果有三页的ViewPager，0,1,2
	那么当你开始移动ViewPager，你当前移动的距离就是:
	(position*ViewPager一页的宽度)+(positionOffsetPixels)
	如果这里你看明白，这个你就知道怎么做了
*/
```



####  代码实现  #####

```java
public class Inject extends FrameLayout implements ViewPager.OnPageChangeListener {
    private ShapeDrawable selectShape;
    private MyVp mViewPager;
    private ShapeDrawable defaultShape;
    private View moveView;

    public Inject(Context context) {
        this(context, null);
    }

    public Inject(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public Inject(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);

        {
          //圆点shape图形
            OvalShape ovalShape = new OvalShape();
            defaultShape = new ShapeDrawable(ovalShape);
            defaultShape.getPaint().setColor(Color.BLACK);
            defaultShape.getPaint().setStyle(Paint.Style.FILL_AND_STROKE);
        }

        {
          //被选中的shape图形
            OvalShape ovalShape = new OvalShape();
            selectShape = new ShapeDrawable(ovalShape);
            selectShape.getPaint().setColor(Color.rgb(0x75, 0xd1, 0xd9));
            selectShape.getPaint().setStyle(Paint.Style.FILL_AND_STROKE);
        }

    }

  //绑定ViewPager，第一个为Viewpager，第二个参数不用理会
    public void setViewPager(MyVp vp, boolean isInfinite) {
        this.mViewPager = vp;
        mViewPager.addOnPageChangeListener(this);
        initView();
    }

    @Override
    protected void onDetachedFromWindow() {
      //释放引用
        if (mViewPager != null)
            mViewPager.removeOnPageChangeListener(this);
        mViewPager = null;
        super.onDetachedFromWindow();
    }

  //初始化point
    private void initView() {
      //不动的点
        for (int i = 0; i < mViewPager.getAdapter().getCount(); i++) {
            View v = new View(getContext());
            v.setBackgroundDrawable(defaultShape);
            LayoutParams layoutParams = new LayoutParams(14, 14);
            layoutParams.leftMargin = i * 42;
            addView(v, layoutParams);
        }


      //移动的点
        moveView = new View(getContext());
        moveView.setBackgroundDrawable(selectShape);
        LayoutParams layoutParams = new LayoutParams(14, 14);
        addView(moveView, layoutParams);
    }

    @Override
    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
      //根据信息计算要移动的点的leftMargin
        LayoutParams layoutParams = (LayoutParams) moveView.getLayoutParams();
        layoutParams.leftMargin = (int) (position * 42 + (42 * positionOffset));
        moveView.setLayoutParams(layoutParams);
    }

    @Override
    public void onPageSelected(int position) {
    }

    @Override
    public void onPageScrollStateChanged(int state) {
    }
}

```
源码下载:[Github](https://github.com/Gloomyer/Simple-ViewPager-Indicator-Demo)
