---
title: RecyclerView ItemTouchHelper 实现上下拖拽排序 滑动删除
categories: Android
date: 2017-04-13 14:27:52
tags:
- recyclerview
- ItemTouchHelper
---
<Excerpt in index | 首页摘要> 
> RecyclerView ItemTouchHelper 实现上下拖拽排序 滑动删除
>
> <!-- more -->
> <The rest of contents | 余下全文>  

#### 概述

不得不说RecyclerView的强大简直无可匹敌。

今天学习到他有一个帮助类，叫做ItemTouchHelper

看名字就可以得知它是帮助我们在item的操作上面提供给我们一些帮助的效果。

item操作主要是两种，一种是拖拽:drag,一种是滑动:swipe



#### 效果图

让我们先来看看效果图:

![](http://gloomyer.com/img/img/itemtouchhelper_demo1.gif)

#### UI代码

通过实践发现这个帮助类相当的强大，而且使用起来也十分的简单与方便。

具体看代码

界面代码:

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/mRecyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</LinearLayout>
```

itemxml代码:

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="60dp"
    android:background="#fff"
    android:orientation="vertical">

    <ImageView
        android:id="@+id/image"
        android:layout_width="48dp"
        android:layout_height="48dp"
        android:layout_gravity="center_vertical"
        android:layout_marginLeft="12dp"
        android:src="@mipmap/ic_launcher" />


    <TextView
        android:id="@+id/nick"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="68dp"
        android:layout_marginTop="6dp"
        android:text="nick"
        android:textSize="16sp" />

    <TextView
        android:id="@+id/desc"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom"
        android:layout_marginBottom="8dp"
        android:layout_marginLeft="68dp"
        android:text="desc"
        android:textSize="15sp" />

    <TextView
        android:id="@+id/time"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="right"
        android:layout_marginRight="8dp"
        android:layout_marginTop="6dp"
        android:text="09:55:31" />
</FrameLayout>
```

这个都十分的简单

看看初始化代码!

```java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initData();//初始化虚假数据
        mRecyclerView = (WrapRecyclerView) findViewById(R.id.mRecyclerView);
        mRecyclerView.setLayoutManager(new LinearLayoutManager(this));
        mRecyclerView.setAdapter(new Adapter());
      	//分割线，无须在意
        //mRecyclerView.addItemDecoration(new CutofRuleItemDecoration(this));
      	//重点就在这里，这个help就是我们的帮助类
      	//MyHelperCall是我们自己的一个自定义回调实现类
      	//具体的重点代码也在这里面
      	//我们还将adapter实现了一个ItemMoveListener接口
      	//是因为我们回调接口需要回调adapter来实现两个功能(交换/移除)
        mHelper = new ItemTouchHelper(new MyHelperCall((ItemMoveListener) mRecyclerView.getAdapter()));
      	//这里将help绑定RecyclerView
        mHelper.attachToRecyclerView(mRecyclerView);
    }


    private void initData() {
        mDatas = new ArrayList<>();
        for (int i = 0; i < 50; i++) {
            mDatas.add(new Bean("nick" + i, "desc" + i));
        }
    }
```



先看下ItemMoveListener接口:

```java
//很简单 一个交换、一个移除
public interface ItemMoveListener {
    void onMove(int srcPos, int targetPos);

    void onRemove(int pos);
}
```



为了模拟数据我们还有一个pojo类 

```java
private class Bean {
        String nick;
        String desc;

        public Bean(String nick, String desc) {
            this.nick = nick;
            this.desc = desc;
        }
    }
```

让我们看下viewHolder

```java
class Holder extends RecyclerView.ViewHolder {

        ImageView image;//图片。我们这里在xml里面写死了
        TextView nick;
        TextView desc;

        public Holder(View itemView) {
            super(itemView);
            image = (ImageView) itemView.findViewById(R.id.image);
            nick = (TextView) itemView.findViewById(R.id.nick);
            desc = (TextView) itemView.findViewById(R.id.desc);
        }
    }
```



#### 具体逻辑代码

先看一下adapter的

```java
class Adapter extends RecyclerView.Adapter<Holder> implements ItemMoveListener {


        @Override
        public Holder onCreateViewHolder(ViewGroup parent, int viewType) {
            View view = LayoutInflater.from(MainActivity.this)
                    .inflate(R.layout.item, parent, false);
            return new Holder(view);
        }

        @Override
        public void onBindViewHolder(final Holder holder, int position) {
            Bean bean = mDatas.get(position);
            holder.nick.setText(bean.nick);
            holder.desc.setText(bean.desc);
          
          	//这里着重说明下，我们除了长按可以进行移动之外、我们还可以主动调用help.startDrag(holder)
          	//开始拖拽操作
            holder.image.setOnTouchListener(new View.OnTouchListener() {
                @Override
                public boolean onTouch(View v, MotionEvent event) {
                    if (event.getAction() == MotionEvent.ACTION_DOWN) {
                        mHelper.startDrag(holder);
                    }
                    return false;
                }
            });
        }

        @Override
        public int getItemCount() {
            return mDatas == null ? 0 : mDatas.size();
        }

  		//回调的交换数据
        @Override
        public void onMove(int srcPos, int targetPos) {
            //交换数据
            Collections.swap(mDatas, srcPos, targetPos);
            notifyItemMoved(srcPos, targetPos);//通知交换了，并且开始执行动画
        }
		//回调的删除数据
        @Override
        public void onRemove(int pos) {
            mDatas.remove(pos);
            notifyItemRemoved(pos);//通知被删除了
        }
    }
```





重点来了

MyHelpCall代码:

实际上也比较简单

```java
public class MyHelperCall extends ItemTouchHelper.Callback {

    private final ItemMoveListener mItemMoveListener;//实现了数据操作的回调的接口

    public MyHelperCall(ItemMoveListener listener) {
        this.mItemMoveListener = listener;
    }

    @Override
    public int getMovementFlags(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder) {
      	//这里 表明了 我们需要接收:
      	//		上下的拖拽操作
      	//		左右的滑动操作
        int dragFlags = ItemTouchHelper.UP | ItemTouchHelper.DOWN;
        int swipeFlags = ItemTouchHelper.LEFT | ItemTouchHelper.RIGHT;
        return makeMovementFlags(dragFlags, swipeFlags);
    }

  	//当move事件发生
    @Override
    public boolean onMove(RecyclerView recyclerView, RecyclerView.ViewHolder srcHolder, RecyclerView.ViewHolder targetHolder) {
      //两个item的viewtype必须一致 可以不用加
        if (srcHolder.getItemViewType() != targetHolder.getItemViewType())
            return false;
      	//回调接口实现数据交换
        mItemMoveListener.onMove(srcHolder.getAdapterPosition(), targetHolder.getAdapterPosition());
        return true;
    }
	
  //当滑动完成之后的回调
    @Override
    public void onSwiped(RecyclerView.ViewHolder viewHolder, int direction) {
      	//通知数据移除
        mItemMoveListener.onRemove(viewHolder.getAdapterPosition());
    }

  	//这个接口是在我们的item被选中的时候
  	//actionState 有三种状态: 闲置、拖拽、滑动
  	//我这里的操作就是当一个item发生了拖拽操作
  	//我们就修改这个item的背景色
  	//让它醒目些
    @Override
    public void onSelectedChanged(RecyclerView.ViewHolder holder, int actionState) {
        if (actionState == ItemTouchHelper.ACTION_STATE_DRAG) {
            holder.itemView.setBackgroundColor(
                    holder.itemView.getContext().getResources().getColor(R.color.selectColor));
        }
        super.onSelectedChanged(holder, actionState);
    }

  	//这个回调是item被选中，正在发生状态的回调
  	//同样有一个actionState 告诉我们正在发生什么操作
  	//我们这里监听滑动、并且通过左右移动的value(dx)值来实现
  	//拖拽删除的时候 让item透明的渐变和缩放
    @Override
    public void onChildDraw(Canvas c, RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder
            , float dX, float dY, int actionState, boolean isCurrentlyActive) {
        super.onChildDraw(c, recyclerView, viewHolder, dX, dY, actionState, isCurrentlyActive);
        if (actionState == ItemTouchHelper.ACTION_STATE_SWIPE) {
            //滑动状态
            Log.e("test", "dx:" + dX + ",dY:" + dY);
            viewHolder.itemView.setAlpha(1 - Math.abs(dX) / viewHolder.itemView.getWidth());
            viewHolder.itemView.setScaleX(1 - Math.abs(dX) / viewHolder.itemView.getWidth());
            viewHolder.itemView.setScaleY(1 - Math.abs(dX) / viewHolder.itemView.getWidth());
        }
    }
	
  //这里是操作完成之后的一个回调。
  	//用于清除我们之前做完一些操作之后的负面影响
  	//如果不在这里清除，会产生一些问题(类似于ListView的checkbox事件影响)
    @Override
    public void clearView(RecyclerView recyclerView, RecyclerView.ViewHolder holder) {
        super.clearView(recyclerView, holder);
        holder.itemView.setBackgroundColor(Color.WHITE);
        holder.itemView.setAlpha(1);
        holder.itemView.setScaleX(1);
        holder.itemView.setScaleY(1);
    }
  
  //还有两个回调
  //这两个回调不需要重写
  //他们的默认返回值是true
  //如果返回了false
  //那么这些操作就需要我们主动调用helper的startDrag(holder)/startSwipe(holder)
  
  
  //是否启用item的滑动
  @Override
    public boolean isItemViewSwipeEnabled() {
        return super.isItemViewSwipeEnabled();
    }

  //是否启用item的长按
    @Override
    public boolean isLongPressDragEnabled() {
        return super.isLongPressDragEnabled();
    }
}
```

