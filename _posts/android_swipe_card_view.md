---
title: Android滑动消失卡片RecyclerView
date: 2018-4-27 13:52:27
categories: Android
tags: 
- Android
---
<Excerpt in index | 首页摘要> 
> Android滑动消失卡片RecyclerView
> 
> <!-- more -->
> <The rest of contents | 余下全文> 

## Demo ##
先看效果图

![](https://gloomyer.com/img/img/android_swpie_card.gif)

## 实现 ##

### 思路 ###
利用Recyclerview来实现

重写LayoutManager 实现item层叠显示

下面的缩放 跟随滑动距离放大。

顶部的正常显示，重写RecyclerView 托管滑动手势管理。

### 代码 ###

LayoutManager
```
import android.support.v7.widget.RecyclerView;
import android.view.View;
import android.view.ViewGroup;

public class SwipeCardLayoutManager extends RecyclerView.LayoutManager {
    @Override
    public RecyclerView.LayoutParams generateDefaultLayoutParams() {
        return new RecyclerView.LayoutParams(
                ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.MATCH_PARENT);
    }

    @Override
    public void onLayoutChildren(RecyclerView.Recycler recycler, RecyclerView.State state) {
        detachAndScrapAttachedViews(recycler);
        for (int i = 0; i < getItemCount(); i++) {
            View child = recycler.getViewForPosition(i);
            measureChildWithMargins(child, 0, 0);
            addView(child);
            int width = getDecoratedMeasuredWidth(child);
            int height = getDecoratedMeasuredHeight(child);
            layoutDecorated(child, 0, 0, width, height);
            //缩放
            if (i < getItemCount() - 1) {
                child.setScaleX(0.8f);
                child.setScaleY(0.8f);
            }
        }
    }
}
```

然后我们再写一个adapter，里面封装删除顶部的方法
ComRvHolder 是通用viewholder代码就不贴了，根据各自项目里面的方式去做就好了

```
import com.miaozan.xpro.common.ComRvHolder;

import java.util.List;

public abstract class SwipeCardAdapter extends RecyclerView.Adapter<ComRvHolder> {
    protected List mList;

    public SwipeCardAdapter(List list) {
        mList = list;
    }

    /**
     * 删除最顶部Item
     */
    public void delTopItem() {
        int position = getItemCount() - 1;
        mList.remove(position);
        notifyItemRemoved(position);
    }

    @Override
    public int getItemCount() {
        return mList == null ? 0 :mList.size();
    }
}

```

重点来了。

RecyclerView


这里说一下，mDecorView 是用activity获取的，我的项目里面有activitymanager 可以拿到 如果你们项目没有 你需要想办法将activity传进view

mTopViewY 是顶部item距离屏幕顶部的距离，这个根据实际情况自己计算

PlayV2BaseInfo是要展示的数据集合，用来判断是否可以继续拖拽消失的，自己根据情况判断好了。
```
import android.animation.Animator;
import android.animation.TimeInterpolator;
import android.animation.ValueAnimator;
import android.annotation.SuppressLint;
import android.content.Context;
import android.graphics.Bitmap;
import android.support.annotation.Nullable;
import android.support.v7.widget.RecyclerView;
import android.util.AttributeSet;
import android.view.MotionEvent;
import android.view.View;
import android.view.animation.LinearInterpolator;
import android.view.animation.OvershootInterpolator;
import android.widget.FrameLayout;
import android.widget.ImageView;

import com.miaozan.xpro.bean.playv2.PlayV2BaseInfo;
import com.miaozan.xpro.manager.ActivityManager;
import com.miaozan.xpro.utils.DensityUtil;

import java.util.List;

public class SwipeCardRecyclerView extends RecyclerView {
    private final static String TAG = "SwipeCardRecyclerView";

    private float mTopViewY;

    private float mTouchDownY;

    private float mBorder;

    private ItemRemovedListener mRemovedListener;

    private FrameLayout mDecorView;
    private int[] mDecorViewLocation = new int[2];
    private List<PlayV2BaseInfo> mDatas;

    public void setData(List<PlayV2BaseInfo> mDatas) {
        this.mDatas = mDatas;
    }


    public interface ItemRemovedListener {
        void onRemove();
    }


    private boolean isRunAnim;


    public SwipeCardRecyclerView(Context context) {
        super(context);
        initView(context);
    }

    public SwipeCardRecyclerView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        initView(context);
    }

    public SwipeCardRecyclerView(Context context, @Nullable AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        initView(context);
    }

    private void initView(Context context) {
        mBorder = DensityUtil.dip2px(context, 120);
        try {
            mDecorView = (FrameLayout) ActivityManager.getInstance().getTopActivity().getWindow().getDecorView();
            mDecorView.getLocationOnScreen(mDecorViewLocation);
        } catch (Exception e) {
            e.printStackTrace();
        }
        mTopViewY = DensityUtil.dip2px(10) + DensityUtil.getSystemStatusHeight() + DensityUtil.getActionBarHeight();
    }

    public void setRemovedListener(ItemRemovedListener listener) {
        mRemovedListener = listener;
    }


    @Override
    public boolean onInterceptTouchEvent(MotionEvent e) {
        if (e.getY() <= (DensityUtil.getActionBarHeight() + DensityUtil.getSystemStatusHeight()))
            return false;
        return super.onInterceptTouchEvent(e);
    }

    @SuppressLint("ClickableViewAccessibility")
    @Override
    public boolean onTouchEvent(MotionEvent e) {
        if (e.getY() <= (DensityUtil.getActionBarHeight() + DensityUtil.getSystemStatusHeight()))
            return false;
        View topView = getChildAt(getChildCount() - 1);
        if (mDatas == null || mDatas.size() <= 1 || isRunAnim) {
            return super.onTouchEvent(e);
        }
        float touchY = e.getY();
        switch (e.getAction()) {
            case MotionEvent.ACTION_DOWN:
                mTouchDownY = touchY;
                break;
            case MotionEvent.ACTION_MOVE:
                float dy = touchY - mTouchDownY;
                float y = mTopViewY + dy;
                if (y <= mTopViewY)
                    topView.setY(y);
                updateNextItem(Math.abs(topView.getY() - mTopViewY) * 0.2 / mBorder + 0.8);
                break;
            case MotionEvent.ACTION_UP:
                mTouchDownY = 0;
                touchUp(topView);
                break;
        }
        return super.onTouchEvent(e);
    }

    /**
     * 更新下一个View的宽高
     *
     * @param factor
     */
    private void updateNextItem(double factor) {
        if (getChildCount() < 2) {
            return;
        }
        if (factor > 1) {
            factor = 1;
        }
        View nextView = getChildAt(getChildCount() - 2);
        nextView.setScaleX((float) factor);
        nextView.setScaleY((float) factor);
    }

    /**
     * 手指抬起时触发动画
     *
     * @param view
     */
    private void touchUp(final View view) {
        isRunAnim = true;
        float targetY;
        boolean del;
        if (Math.abs(view.getY() - mTopViewY) < mBorder) {
            //不满足条件 恢复
            targetY = mTopViewY;
            del = false;
        } else {
            //满足条件 删除
            del = true;
            targetY = -view.getHeight();
            if (mRemovedListener != null) {
                mRemovedListener.onRemove();
            }
        }
        View animView;
        TimeInterpolator interpolator;
        if (del) {
            animView = getMirrorView(view);
            interpolator = new LinearInterpolator();
        } else {
            animView = view;
            interpolator = new OvershootInterpolator();
        }
        ValueAnimator va = ValueAnimator.ofFloat(animView.getTop(), targetY)
                .setDuration(300);
        va.setInterpolator(interpolator);
        va.addUpdateListener(animation -> {
            float value = (float) animation.getAnimatedValue();
            animView.setY(value);
            if (!del) {
                updateNextItem(Math.abs(view.getY() - mTopViewY) * 0.2 / mBorder + 0.8);
            }
        });
        va.addListener(new Animator.AnimatorListener() {
            @Override
            public void onAnimationStart(Animator animation) {
            }

            @Override
            public void onAnimationEnd(Animator animation) {
                if (del) {
                    try {
                        mDecorView.removeView(animView);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
                isRunAnim = false;
            }

            @Override
            public void onAnimationCancel(Animator animation) {
            }

            @Override
            public void onAnimationRepeat(Animator animation) {
            }
        });
        va.start();
    }

    /**
     * 创建镜像View替代原有顶部View展示删除动画
     *
     * @param view
     * @return
     */
    private ImageView getMirrorView(View view) {
        view.destroyDrawingCache();
        view.setDrawingCacheEnabled(true);
        final ImageView mirrorView = new ImageView(getContext());
        Bitmap bitmap = Bitmap.createBitmap(view.getDrawingCache());
        mirrorView.setImageBitmap(bitmap);
        view.setDrawingCacheEnabled(false);
        FrameLayout.LayoutParams params = new FrameLayout.LayoutParams(bitmap.getWidth(), bitmap.getHeight());
        int[] locations = new int[2];
        view.getLocationOnScreen(locations);

        mirrorView.setAlpha(view.getAlpha());
        view.setVisibility(GONE);
        ((SwipeCardAdapter) getAdapter()).delTopItem();
        mirrorView.setX(locations[0] - mDecorViewLocation[0]);
        mirrorView.setY(locations[1] - mDecorViewLocation[1]);
        mDecorView.addView(mirrorView, params);
        return mirrorView;
    }
}

```

### 重点使用说明  ###
这样叠加出来的view是相反的，即List最后面的在最上面，业务层处理逻辑要相当注意！新卡片如果是在最底下显示，应该使用insert index应该是0！

还有需要注意的是，这样写layoutmanager，所有的item都会加载出来！不会复用！所以不要让list中包含太多数据！

如果确实需要很大的数据量！那么你需要去对LayoutManager做进一步优化。复用item！