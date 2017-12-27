# IpsmapSDK-Android

[![license](https://img.shields.io/hexpm/l/plug.svg)](https://raw.githubusercontent.com/typ0520/fastdex/master/LICENSE)
[![Download](https://api.bintray.com/packages/xun/maven/com.ipsmap/images/download.svg) ](https://bintray.com/xun/maven/com.ipsmap/_latestVersion)
[![API](https://img.shields.io/badge/API-18%2B-green.svg?style=flat)](https://android-arsenal.com/api?level=18)
[![Contact](https://img.shields.io/badge/Author-IpsMap-orange.svg?style=flat)](http://ipsmap.com)

IpsmapSDK-Android 是一套基于 Android 4.3 及以上版本的室内地图应用程序开发接口，供开发者在自己的Android应用中加入室内地图相关的功能，包括：地图显示（多楼层、多栋楼）、室内导航、模拟导航、语音播报等功能。

## 获取AppKey和MapId
请联系dev@ipsmap.com

## 添加依赖

```
compile ('com.ipsmap:ipsmap:1.3.6', {
        exclude group: 'com.android.support'
    })
```
如果仅仅使用定位模块请参考ipslocation demo README


## 目前支持的cpu 架构 arm,暂时不支持其他架构,请配置下面的cpu架构
```
ndk {
            // 设置支持的 SO 库构架
            abiFilters 'armeabi'
}
```
## 加入权限
导入IpsmapSDK后需要
```
 
    
     <!-- 注意新增的权限,注意添加-->
        <uses-permission android:name="android.permission.VIBRATE" />
         <!-- sdk 使用需要的权限 -->
         <!-- if use wifi indoor positioning -->
         <uses-permission android:name="android.permission.INTERNET" />
         <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
         <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
         <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
         <!-- if use ble indoor positioning -->
         <uses-permission android:name="android.permission.BLUETOOTH" />
         <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
         <!-- general permission -->
         <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
         <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
         <!-- 连接网络权限，用于执行云端语音能力 -->
         <!-- 获取手机录音机使用权限，听写、识别、语义理解需要用到此权限 -->
         <uses-permission android:name="android.permission.RECORD_AUDIO" />
         <!-- 读取网络信息状态 -->
         <!-- 允许程序改变网络连接状态 -->
         <uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
         <!-- 读取手机信息权限 -->
         <uses-permission android:name="android.permission.READ_PHONE_STATE" />
         <!-- 读取联系人权限，上传联系人需要用到此权限 -->
         <uses-permission android:name="android.permission.READ_CONTACTS" />
         <!-- 外存储写权限，构建语法需要用到此权限 -->
         <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
         <!-- 外存储读权限，构建语法需要用到此权限 -->
         <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
         <!-- 配置权限，用来记录应用配置信息 -->
         <uses-permission android:name="android.permission.WRITE_SETTINGS" />
         <uses-permission android:name="android.permission.READ_LOGS" />
         <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
         <uses-permission android:name="android.permission.REQUEST_IGNORE_BATTERY_OPTIMIZATIONS" />
         <!-- 判断程序是否在前台运行,必须 -->
         <uses-permission android:name="android.permission.GET_TASKS" />
         <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS" />
         <uses-permission android:name="android.permission.VIBRATE" />
```

## 使用
初始化

在Application 的onCreate 方法中进行初始化
``` 
    使用默认配置信息
    IpsMapSDK.init(context, IPSMAP_APP_KEY);
    或
    定制配置信息 ,使用微信分享功能请实现相关的接口
    IpsMapSDK.init(new IpsMapSDK.Configuration.Builder(context)
                .appKey(Constants.IPSMAP_APP_KEY)
                .shareToWechatListener(this)
                //正式版请关闭 默认是关闭的
                .debug(false)
                .build());
                

```
SDK内部实现了分享功能，使用的前提是需要申请微信的appkey，并且需要实现接口ShareToWechatListener接口
参考代码如下：
```

    参考代码
   @Override
    public void shareToWechat(String url, String title, String description, Bitmap bitmap) {
        try {
            IWXAPI wxApi = WXAPIFactory.createWXAPI(this, "YOUR WECHAT APP_ID");
            wxApi.registerApp("YOUR WECHAT APP_ID");
            if (!wxApi.isWXAppInstalled()) {
                Toast.makeText(this, "未安装微信", Toast.LENGTH_SHORT).show();
                return;
            }
            WXWebpageObject webpage = new WXWebpageObject();
            webpage.webpageUrl = url;
            WXMediaMessage msg = new WXMediaMessage(webpage);
            msg.title = title;
            msg.description = description;
            msg.setThumbImage(bitmap);
            SendMessageToWX.Req req = new SendMessageToWX.Req();
            req.transaction = buildTransaction("webpage");
            req.message = msg;
            req.scene = SendMessageToWX.Req.WXSceneSession;
            wxApi.sendReq(req);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private String buildTransaction(final String type) {
        return (type == null) ? String.valueOf(System.currentTimeMillis()) : type + System.currentTimeMillis();
    }
                

```

```
将微信分享通过浏览器打开的acitivty 中加入配置 ,建议新建一个界面,不要现有的逻辑冲突.
这个界面的功能一个中转的功能,是通过浏览器唤起这个界面,这个界面打开地图.
<!--微信分享-->
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data
            android:host="share"
            android:scheme=你的scheme></data>
    </intent-filter>
<!--微信分享结束-->

重写以下两个方法
  @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_share);
        //如果不是新建的页面判断一下scheme
        new Handler().postDelayed(new Runnable() {
            @Override
            public void run() {
                IpsMapSDK.shareLinkToMapView(getIntent());
                finish();
            }
        }, 500);
    }

    @Override
    protected void onNewIntent(Intent intent) {
        super.onNewIntent(intent);
        IpsMapSDK.shareLinkToMapView(intent);
        finish();
    }


```



启动地图
```
IpsMapSDK.openIpsMapActivity(context, map_id);
或
IpsMapSDK.openIpsMapActivity(context, map_id, target_id);
```

定位监听,获取当前的位置,可以参考ipslocation demo ,需要提前获取定位和蓝牙权限
```
ipsClient = new IpsClient(context, map_id); 
ipsClient.registerLocationListener(new IpsLocationListener() {
    @Override
    public void onReceiveLocation(IpsLocation ipsLocation){
    if(ipsLocation == null){
        //定位失败;
        return;
    }
    //是否在Map内
    ipsLocation.isInThisMap()

    }
});
ipsClient.start();
```

activity 结束时调用
```
@Override
protected void onDestroy() {
    super.onDestroy();
    ipsClient.stop();
}
```

## 混淆
```
-dontwarn com.baidu.**
-keep class com.baidu.** {*;}
-dontwarn com.iflytek.**
-keep class com.iflytek.**{*;}
-keep public class com.sails.engine.patterns.IconPatterns
```

微信分享以及复制跳转请参考demo

## FAQ
1.0
![](/pic/7991511168017_.pic.jpg)
![](/pic/8021511168507_.pic.jpg)
出现上面的类似xml资源文件缺失的情况:
两种解决方案:
1. 在通过gradle 引用是加入exclude group: 'com.android.support' ,并且自己加入compile 'com.android.support:appcompat-v7:版本号'
建议方式.建议版本号25.3.1
2. 修改项目的support 支持和  compile 'com.android.support:appcompat-v7:25.3.1' 版本号一致

2.0 
```
app如果使用了okhttp ,glide ...出现第三发开源库 冲突
两种解决方案:
1.通过  exclude group: "com.squareup.okhttp3" 方式处理
然后保留项目的okhttp和glide 
2.保持和sdk的一致引入的第三方库版本号一致.否则有可能出现冲突
```
```
"glide"             : "com.github.bumptech.glide:glide:3.7.0",
"okhttp"            : "com.squareup.okhttp3:okhttp:3.8.0",
"gson"              : "com.google.code.gson:gson:2.8.2",
 ```        


 3.0
 
![](/pic/AC0BDB3E-C313-4644-AB5F-F3C8FA209AEC.png) 
```


    allprojects {
        repositories {
            jcenter()
            maven { url "https://jitpack.io" }
            flatDir {
                dirs 'libs'
            }
        }
    }
    
    compileOptions {
         sourceCompatibility JavaVersion.VERSION_1_8
         targetCompatibility JavaVersion.VERSION_1_8
     }
```


