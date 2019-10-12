---
title: 如何正确的创建正方形ImageVIew
date: 2017-10-27 15:27:40
categories: Android
tags: 
- 自定义ViewModel
---
<Excerpt in index | 首页摘要> 
> 如何正确的创建正方形ImageVIew
> <!-- more -->
> <The rest of contents | 余下全文> 

经常在开发任务中碰见类似的要求:

正方形的ImageVIew，一行多少个，每个占据屏幕的1/xx(3、4)。

其实宽度好说 ，主要是高度不好设定。所以过去我的做法是读取当前屏幕的宽度 然后计算出合适的宽高给ImageVIew。

类似如下:
```
View view =
    LayoutInflater.from(PreAct.this)
            .inflate(R.layout.gloomy_item_preview, parent, false);
if (size == 0) {
size = Utils.getScreenWidth(PreAct.this) / 4 - 16;
}

ViewGroup.LayoutParams layoutParams = view.getLayoutParams();
layoutParams.height = size;
layoutParams.width = size;
view.setLayoutParams(layoutParams);
return new Holder(view);
```

今天波波给我讲了个更简单的方法.现在想来真是简单。

布局中用到的ImageView用自定义的.

然后代码如下:

```
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	super.onMeasure(widthMeasureSpec, widthMeasureSpec);
}
```

然后ImageView的宽高直接match_parent即可.

具体的交给GridLayoutManager去就好了..
