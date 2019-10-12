---
title: 华为PUSH自定义行为Intent的坑
date: 2018-9-5 20:34:05
categories: Android
tags:
- Android
- HMS
---
<Excerpt in index | 首页摘要> 
> 华为PUSH自定义行为Intent的坑
>
<!-- more -->
<The rest of contents | 余下全文>  
  
## 概要

目前，工信部统一推送渠道没音的情况下，APP想要尽可能将内容推送到用户手机上，只有接入多渠道推送来实现。

其他的姑且不谈，没啥坑，按标准照做即可。

华为的是个大坑，溅一脸血的那种。

## 客户端

首先，需要你注册一个透明的代理Activity（为了区分其他渠道 华为的我们自己单独处理一层）

```
<activity
    android:name=".model.push.HuaweiPushAct"
    android:theme="@android:style/Theme.Translucent">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <data
            android:host="com.miaozan.xpro"
            android:path="/push_detail"
            android:scheme="hwpush_scheme" />
    </intent-filter>
</activity>
```
host:包名

path:路径

scheme:渠道 这三个是自定义的 按需配置即可

---

```
public class HuaweiPushAct extends Activity {
    private static final String TAG = "HuaweiPushAct";

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,WindowManager.LayoutParams.FLAG_FULLSCREEN);
    }

	@Override
    protected void onResume() {
        super.onResume();
        Intent intent = getIntent();
        Intent startIntent = new Intent(this, V3MainActivity.class);
        try {
            String params = intent.toUri(Intent.URI_INTENT_SCHEME);
            //Loger.E(TAG, "params:" + params);
            params = params.substring(params.indexOf("push_detail?") + 12, params.lastIndexOf("#Intent;"));
            Loger.E(TAG, "params:" + params);
            String[] parames = params.split("&");
            for (int i = 0; i < parames.length; i++) {
                String[] temp = parames[i].split("=");
                if(temp[0].equals("type")){
                    startIntent.putExtra(temp[0], Integer.parseInt(temp[1]));
                }else if (temp[0].equals("targetUserId")){
                    startIntent.putExtra(temp[0], Long.parseLong(temp[1]));
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        startActivity(startIntent);
        finish();
    }
}
```

就是一个透明的中介Activity

onResume中解析所需要的参数

---

注册完成，生成服务端需要的intent字符串

```
Intent intent = new Intent(Intent.ACTION_VIEW);
intent.setData(Uri.parse("hwpush_scheme://com.miaozan.xpro/push_detail?type=2&toUserId=1"));
intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
String intentUri = intent.toUri(Intent.URI_INTENT_SCHEME);
Log.e("hwpush", "intnetUri=    " + intentUri);
```

hwpush_scheme:manifest文件中注册act的时候的scheme字段后面的

com.miaozan.xpro:manifest文件中注册act的时候的host字段后面的

/push_detail:manifest文件中注册act的时候的path字段后面的

type=2&toUserId=1 自定义行为需要携带的参数，这里有两个多个用&连接即可

生成的字符串类似如下

```
hwpush_scheme://com.miaozan.xpro/push_detail?type=2&targetUserId=1#Intent;launchFlags=0x10000000;end
```

然后就可以把这个字符串交给服务端了


## 服务端

服务端比较简单

依赖Gson库（json格式化需求）

直接看代码(不含申请access_token)

```
import com.google.gson.JsonArray;
import com.google.gson.JsonObject;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.net.HttpURLConnection;
import java.net.URL;
import java.net.URLEncoder;
import java.text.MessageFormat;

public class Main {

    private static String appId = "";//用户在华为开发者联盟申请的appId和appSecret（会员中心->应用管理，点击应用名称的链接）

    private static String tokenUrl = "https://login.cloud.huawei.com/oauth2/v2/token"; //获取认证Token的URL

    private static String apiUrl = "https://api.push.hicloud.com/pushsend.do"; //应用级消息下发API

    private static String accessToken = "";//下发通知消息的认证Token  

    public static void main(String[] args) throws Exception {


        JsonArray deviceTokens = new JsonArray();
        deviceTokens.add(""); //设备ID，可以为多个

        JsonObject body = new JsonObject();//仅通知栏消息需要设置标题和内容，透传消息key和value为用户自定义
        body.addProperty("title", "Push message title");//消息标题
        body.addProperty("content", "Push message content");//消息内容体

        JsonObject param = new JsonObject();

        param.addProperty("intent", "hwpush_scheme://com.miaozan.xpro/push_detail?type=2&targetUserId=1#Intent;launchFlags=0x10000000;end");
		//这里需要将type=2 targetUserId=1 替换成自己所需要的			

        JsonObject action = new JsonObject();
        action.addProperty("type", 1);//1:自定义行为
        action.add("param", param);//消息点击动作参数

        JsonObject msg = new JsonObject();
        msg.addProperty("type", 3);//3: 通知栏消息，异步透传消息请根据接口文档设置
        msg.add("body", body);//通知栏消息body内容
        msg.add("action", action);//消息点击动作

        JsonObject ext = new JsonObject();//扩展信息，含BI消息统计，特定展示风格，消息折叠。
        ext.addProperty("biTag", "Trump");//设置消息标签，如果带了这个标签，会在回执中推送给CP用于检测某种类型消息的到达率和状态

        JsonObject hps = new JsonObject();//华为PUSH消息总结构体
        hps.add("msg", msg);
        hps.add("ext", ext);

        JsonObject payload = new JsonObject();
        payload.add("hps", hps);
        System.out.println(payload.toString());
        String postBody = MessageFormat.format(
                "access_token={0}&nsp_svc={1}&nsp_ts={2}&device_token_list={3}&payload={4}",
                URLEncoder.encode(accessToken, "UTF-8"),
                URLEncoder.encode("openpush.message.api.send", "UTF-8"),
                URLEncoder.encode(String.valueOf(System.currentTimeMillis() / 1000), "UTF-8"),
                URLEncoder.encode(deviceTokens.toString(), "UTF-8"),
                URLEncoder.encode(payload.toString(), "UTF-8"));

        String postUrl = apiUrl + "?nsp_ctx=" + URLEncoder.encode("{\"ver\":\"1\", \"appId\":\"" + appId + "\"}", "UTF-8");

		//这里用的urlconnection 发送的post请求，可以根据需要自行修改
        System.out.println(doJsonPost(postUrl, postBody));
    }

    private static String doJsonPost(String urlPath, String Json) {
        // HttpClient 6.0被抛弃了
        String result = "";
        BufferedReader reader = null;
        try {
            URL url = new URL(urlPath);
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            conn.setRequestMethod("POST");
            conn.setDoOutput(true);
            conn.setDoInput(true);
            conn.setUseCaches(false);
            conn.setRequestProperty("Connection", "Keep-Alive");
            conn.setRequestProperty("Charset", "UTF-8");
            // 设置文件类型:
            conn.setRequestProperty("Content-Type", "application/json; charset=UTF-8");
            // 设置接收类型否则返回415错误
            //conn.setRequestProperty("accept","*/*")此处为暴力方法设置接受所有类型，以此来防范返回415;
            conn.setRequestProperty("accept", "application/json");
            // 往服务器里面发送数据
            if (Json != null && !"".equalsIgnoreCase(Json)) {
                byte[] writebytes = Json.getBytes();
                // 设置文件长度
                conn.setRequestProperty("Content-Length", String.valueOf(writebytes.length));
                OutputStream outwritestream = conn.getOutputStream();
                outwritestream.write(Json.getBytes());
                outwritestream.flush();
                outwritestream.close();
                System.out.println("hlhupload" + "doJsonPost: conn" + conn.getResponseCode());
            }
            if (conn.getResponseCode() == 200) {
                reader = new BufferedReader(
                        new InputStreamReader(conn.getInputStream()));
                result = reader.readLine();
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (reader != null) {
                try {
                    reader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return result;
    }
}

```