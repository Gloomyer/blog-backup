---
title: Android Volley网络框架的几个坑 
date: 2016-10-25 09:58:01
categories: Android
tags: 
- volley
- 乱码
---
<Excerpt in index | 首页摘要> 
> Android解决Volley乱码问题
>
> JsonObjectRequest Post提交参数提交不上去
>
> 请求创建封装库
> <!-- more -->
> <The rest of contents | 余下全文> 

##  引言  ##

各种原因，现在项目网络框架决定从okhttp切换至volley。

然后自己封装了一个简单的封装库和解决乱码问题，还有JsonRequest重写getParams()无效的一个替代解决方案.



##  StringRequest乱码问题  ##

先要知道乱码问题出在哪里。

我们常用编码是UTF-8.而Volley默认的解码方式是根据http协议头的编码来解码的。

一种方式是要求后台在所有请求的返回头部添加charset=utf8,但是这就有点麻烦后台了。

我们是要自立更生的！

我们看下Volley的StringRequest源码.

```java
@Override
protected Response<String> parseNetworkResponse(NetworkResponse response) {
  String parsed;
  try {
    //重点就在这里
    parsed = new String(response.data, HttpHeaderParser.parseCharset(response.headers));
  } catch (UnsupportedEncodingException e) {
    parsed = new String(response.data);
  }
  return Response.success(parsed, HttpHeaderParser.parseCacheHeaders(response));
}
```

如果你是直接导入的Volley源码的方式，那么找到StringRequest，然后修改这里修改为:

```java
@Override
protected Response<String> parseNetworkResponse(NetworkResponse response) {
  String parsed;
  try {
    parsed = new String(response.data, "UTF-8");
  } catch (UnsupportedEncodingException e) {
    parsed = new String(response.data);
  }
  return Response.success(parsed, HttpHeaderParser.parseCacheHeaders(response));
}
```

如果你是直接通过Gradle导入的，那么你需要创建一个类继承自StringRequest然后重写这个方法。修改成这样。



##  JsonObjectRequest Post提交参数提交不上去  ##

如果你使用JsonObjectRequest你会发现重写getParams()方法，提交参数服务器那边是收不到的。

除非你的服务器是支持Json格式的提交，当一般都不会支持。

那你你需要重写两个方法!

第一种方式.继承自JsonObjectRequest 或者 直接重写、改写为如下的方法

```java
@Override  
public String getBodyContentType() {  
  return "application/x-www-form-urlencoded; charset=" + getParamsEncoding();  
}  

@Override
protected Response<String> parseNetworkResponse(NetworkResponse response) {
  String parsed;
  try {
    parsed = new String(response.data, "UTF-8");
  } catch (UnsupportedEncodingException e) {
    parsed = new String(response.data);
  }
  return Response.success(parsed, HttpHeaderParser.parseCacheHeaders(response));
}

@Override
public byte[] getBody() {
  String mRequestBody = appendParameter(NetConstant.AUTO_ROLL_IMG, map);
  try {
    return mRequestBody == null ? null : mRequestBody.getBytes(PROTOCOL_CHARSET);
  } catch (UnsupportedEncodingException uee) {
    VolleyLog.wtf("Unsupported Encoding while trying to get the bytes of %s using %s",
                  mRequestBody, PROTOCOL_CHARSET);
    return null;
  }
}

private String appendParameter(String url, Map<String, String> params) {
  Uri uri = Uri.parse(url);
  Uri.Builder builder = uri.buildUpon();
  for (Map.Entry<String, String> entry : params.entrySet()) {
    builder.appendQueryParameter(entry.getKey(), entry.getValue());
  }
  return builder.build().getQuery();
}

```



当然，还有第二种方式，就是自己写一个，我这里是继承自之前改写的StringRequest的StrRequest写的

```java
package com.qingxiang.ui.common.volley;


import com.android.volley.Response;
import com.android.volley.VolleyError;

import org.json.JSONException;
import org.json.JSONObject;


public class JsonRequest extends StrRequest {

  private Response.Listener<JSONObject> mListener;

  public JsonRequest(int method, String url, Response.Listener<JSONObject> listener, Response.ErrorListener errorListener) {
    super(method, url, null, errorListener);
    mListener = listener;
  }

  public JsonRequest(String url, Response.Listener<JSONObject> listener, Response.ErrorListener errorListener) {
    super(url, null, errorListener);
    mListener = listener;
  }

  @Override
  protected void deliverResponse(String response) {
    try {
      JSONObject obj = new JSONObject(response);
      mListener.onResponse(obj);
    } catch (JSONException e) {
      deliverError(new VolleyError("Json转换异常"));
    }
  }
}

```



##  构造Request的工具类  ##

```java
package com.qingxiang.ui.common.volley;

import com.android.volley.AuthFailureError;
import com.android.volley.Request;
import com.android.volley.Response;

import org.json.JSONObject;

import java.util.HashMap;
import java.util.Map;
import java.util.Set;

/**
 * Created by Gloomy on 2016/10/25.
 */

public class VU {
    private String mUrl;
    private int mMethod;
    private Object mTag;
    private Map<String, String> mParams;
    private Response.Listener<JSONObject> mRespListener;
    private Response.ErrorListener mErrorListener;

    /**
     * 私有化构造参数
     *
     * @param url
     */
    private VU(String url) {
        mUrl = url;
        mParams = new HashMap<>();
    }

    public static VU get(String url) {
        VU vu = new VU(url);
        vu.mMethod = Request.Method.GET;
        return vu;
    }

    public static VU post(String url) {
        VU vu = new VU(url);
        vu.mMethod = Request.Method.POST;
        return vu;
    }

    public VU TAG(Object tag) {
        mTag = tag;
        return this;
    }

    public VU addParams(String key, String value) {
        mParams.put(key, value);
        return this;
    }

    public VU respListener(Response.Listener<JSONObject> listener) {
        mRespListener = listener;
        return this;
    }

    public VU errorListener(Response.ErrorListener listener) {
        mErrorListener = listener;
        return this;
    }

    public JsonRequest build() {
        if (mMethod == Request.Method.GET) {
            StringBuffer sb = new StringBuffer(mUrl);
            sb.append("?");
            Set<String> keys = mParams.keySet();
            int size = 0;
            for (String key : keys) {
                sb.append(key);
                sb.append("=");
                sb.append(mParams.get(key));
                if (size < (keys.size() - 1))
                    sb.append("&");
                size++;
            }
            mUrl = sb.toString();
        }

        JsonRequest request = new JsonRequest(mMethod, mUrl, mRespListener, mErrorListener) {
            @Override
            protected Map<String, String> getParams() throws AuthFailureError {
                return mParams;
            }
        };
        if (mTag != null)
            request.setTag(mTag);
        return request;
    }
}

```

