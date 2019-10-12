---
title: android-高亮url-text-view
date: 2017-03-28 14:01:37
categories: Android
tags:
- url高亮
---
<Excerpt in index | 首页摘要> 
> 抄微博，
>
> 高亮显示url并响应点击事件
>
> <!-- more -->
> <The rest of contents | 余下全文>  



####  概述  ####

app中有个概念叫做‘阶段’

类似于 微博的一条‘微博’

qq空间、微信的一条‘说说’

关于这个内容的展示。



最近要改成微博的形式。

即 最多显示144个字符。

如果有超出部分在文字末尾显示全文并且相应点击事件。

<font color='red'>如果存在url,高亮，并且相应点击事件</font>

如图:

![](http://gloomyer.com/img/img/stretch-text-view-png.png)



####  实现思路  ####

遍历完整的文本，

获取所有的url和每个url的start和end

然后遍历要显示的内容。

设置相应的span。



####  具体代码  ####

```java
public class StretchTextView extends android.support.v7.widget.AppCompatTextView {
    public static final String TAG = "StretchTextView";

    private boolean hasMulti;
    private OnClickListener fullTextClickListener;
    private String fullText;

    public StretchTextView(Context context) {
        this(context, null);
    }

    public StretchTextView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public StretchTextView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    private void init() {
        setTextColor(getResources().getColor(R.color.darkGray)); // text color
        setTextSize(TypedValue.COMPLEX_UNIT_SP, 14); //text size 
        setLineSpacing(DensityUtils.dp2px(getContext(), 4), 1.0f);//line space 
        setMovementMethod(LinkMovementMethod.getInstance());
    }


    /**
     * 设置描述文本,
     *
     * @param text                  阶段内容
     * @param fullTextClickListener 如果存在全文按钮，点击后的跳转逻辑
     */
  	//主要逻辑，暂不支持xml写 text，请通过代码调用
    public void setString(String text, OnClickListener fullTextClickListener) {
        if (TextUtils.isEmpty(text)) {
            setText("");
            setVisibility(GONE); //内部逻辑需要，如果内容为空当前的textview 不显示
            return;
        }
        setVisibility(VISIBLE);
        this.fullTextClickListener = fullTextClickListener;//点击普通文字和文字末尾的全文跳转逻辑
        this.fullText = text;
        //计算多少个字
      	//我们的字数是不超过144个单位，一个汉字算一个单位，其它算半个！
        double count = 0;
        int lineCount = 0;
        StringBuffer sb = new StringBuffer();
        for (int i = 0; i < text.length(); i++) {
            char ch = text.charAt(i);
            if (isChinese(ch))
                count += 1.0f;
            else
                count += 0.5f;

            sb.append(ch);//加入要展示的数据集中去

            if (count >= 144)
                break;

            if (isNewLinsChar(ch))
                lineCount++;
            if (lineCount > 4)//如果包含4个以上的空格，也不继续展示了
                break;

        }
		//如果最后一个字符是换行，去除
        if (isNewLinsChar(sb.charAt(sb.length() - 1)))
            sb = sb.deleteCharAt(sb.length() - 1);

        //判断是否满足显示全文的条件
        hasMulti = count > 144;
        if (!hasMulti)
            hasMulti = lineCount > 4;

        if (hasMulti)
            sb.append("... 全文");//如果包含未能展示完的文字
        mySetText(sb.toString());
    }


    private void mySetText(String text) {
        setText(getClickableSpan(text));
    }

    private SpannableString getClickableSpan(String text) {

        SpannableString spannableInfo
                = new SpannableString(text);

        //全文按钮的点击
        if (hasMulti) {
            spannableInfo.setSpan(new Clickable(fullTextClickListener, R.color.blue),
                    text.length() - 2,
                    text.length(),
                    Spanned.SPAN_EXCLUSIVE_EXCLUSIVE
            );
        }
        //找到所有的url
        List<UrlTarget> urlTargets = myGetUrls();

        //处理点击逻辑
        if (urlTargets.isEmpty())
            spannableInfo.setSpan(new Clickable(fullTextClickListener, R.color.darkGray),
                    0,
                    hasMulti ? text.length() - 6 : text.length(),
                    Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
        else {
            //处理包含url连接的情况
            int length = hasMulti ? text.length() - 6 : text.length();
            //length
            int lastNormalStart = 0;
            int lastEnd = 0;
            for (int i = 0; i <= length; i++) {
                boolean isurl = false;
                for (UrlTarget target : urlTargets) {
                    if (target.has(i)) {
                        /**
                         * 处理上一次的luoji
                         */
                        if (i > 0)
                            spannableInfo.setSpan(
                                    new Clickable(fullTextClickListener, R.color.darkGray),
                                    lastNormalStart,
                                    i,
                                    Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);

                        if (!target.isHandle) {
                            spannableInfo.setSpan(
                                    new Clickable(v -> {
                                        WebActivity.startActivity(getContext(), target.url);
                                    }, R.color.blue),
                                    target.start,
                                    (target.end >= length ? length : target.end),
                                    Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
                            target.isHandle = true;
                            lastNormalStart = target.end;
                            lastEnd = target.end;
                            i = lastNormalStart;
                        }
                        break;
                    }
                }
            }
            //处理最后一次逻辑
            //Log.e(TAG, "lastEnd:" + lastEnd);
            if (lastEnd < length) {
                spannableInfo.setSpan(new Clickable(fullTextClickListener, R.color.darkGray),
                        lastEnd,
                        length,
                        Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
            }

        }

        return spannableInfo;
    }

    private class UrlTarget {
        String url;
        int start;
        int end;
        boolean isHandle;

        /**
         * 具体的字符位置是否包含在当前的url位置中
         *
         * @param location
         * @return
         */
        private boolean has(int location) {
            return location >= start && location <= end;
        }

        @Override
        public String toString() {
            return "UrlTarget{" +
                    "url='" + url + '\'' +
                    ", start=" + start +
                    ", end=" + end +
                    '}';
        }
    }

    private class Clickable extends ClickableSpan implements View.OnClickListener {
        private final int textColor;
        private View.OnClickListener mListener;

        public Clickable(View.OnClickListener l, int color) {
            mListener = l;
            textColor = color;
        }

        @Override
        public void onClick(View v) {
            mListener.onClick(v);
            ((TextView) v).setHighlightColor(Color.argb(0, 0, 0, 0));
        }

        @Override
        public void updateDrawState(TextPaint ds) {
            ds.setColor(getResources().getColor(textColor));    //设置昵称的变色
            ds.setUnderlineText(false);    //去除超链接的下划线
        }
    }

    private List<UrlTarget> myGetUrls() {
        List<UrlTarget> urlTargets = new ArrayList<>();
        String fullText = this.fullText;
        for (; ; ) {
            String url = getCompleteUrl(fullText);
            if (TextUtils.isEmpty(url)) {
                break;
            }
            //data.add(url);
            int start = fullText.indexOf(url);
            fullText = fullText.substring(start + url.length());

            UrlTarget urlTarget = new UrlTarget();
            //start
            if (!urlTargets.isEmpty()) {
                UrlTarget lastUrlTarget = urlTargets.get(urlTargets.size() - 1);
                start = lastUrlTarget.end + start;
            }

            urlTarget.url = url;
            urlTarget.start = start;
            urlTarget.end = start + url.length();
            urlTargets.add(urlTarget);
        }

        Log.e(TAG, urlTargets.toString());
        return urlTargets;
    }

    private static boolean isChinese(char c) {
        Character.UnicodeBlock ub = Character.UnicodeBlock.of(c);
        if (ub == Character.UnicodeBlock.CJK_UNIFIED_IDEOGRAPHS || ub == Character.UnicodeBlock.CJK_COMPATIBILITY_IDEOGRAPHS
                || ub == Character.UnicodeBlock.CJK_UNIFIED_IDEOGRAPHS_EXTENSION_A || ub == Character.UnicodeBlock.CJK_UNIFIED_IDEOGRAPHS_EXTENSION_B
                || ub == Character.UnicodeBlock.CJK_SYMBOLS_AND_PUNCTUATION || ub == Character.UnicodeBlock.HALFWIDTH_AND_FULLWIDTH_FORMS
                || ub == Character.UnicodeBlock.GENERAL_PUNCTUATION) {
            return true;
        }
        return false;
    }

    private static boolean isNewLinsChar(char c) {
        return c == '\r' || c == '\n';
    }

    private static String getCompleteUrl(String text) {
        Pattern p = Pattern.compile("((http|ftp|https)://)(([a-zA-Z0-9\\._-]+\\.[a-zA-Z]{2,6})|([0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}))(:[0-9]{1,4})*(/[a-zA-Z0-9\\&%_\\./-~-]*)?", Pattern.CASE_INSENSITIVE);
        Matcher matcher = p.matcher(text);
        if (matcher.find()) {
            return matcher.group();
        }
        return "";
    }
}
```

