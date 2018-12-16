---
layout:     post
title:      HelperSDK接入文档
subtitle:   SDKDoc of CloudCard
date:       2018-11-24
author:     宸笙
catalog: true
tags:
    - SDKDocs
    
---


# HelperSDK
## 关于HelperSDK
``HelperSDK``是一个帮你封装好了Google登录，GooglePay和AppsFlyer的一个辅助SDK，封装了琐碎的API调用和aidl文件等；
## 接入(集成HelperSDK)
假设您使用的是``AndroidStudio``开发环境，拿到的是``HelperSDKV1.0``的jar包；
1. ``AndroidStudio``的libs配置；
* 在app模块的``build.gradle``文件中，添加libs文件夹配置，如图：
![image](http://bmob-cdn-20286.b0.upaiyun.com/2018/08/15/d1462dd9409b8467808d6a78e120373a.png)
* 点击``Sync Now``后可以看到左侧Android视图下多出了``jniLibs``文件夹，如图：
![image](http://bmob-cdn-20286.b0.upaiyun.com/2018/08/15/ccec58d540a55d40800b6736895492e4.png)
* 将``HelperSDKV1.0``的jar包复制到``jniLibs``文件夹下;
2. 集成``HelperSDK``所依赖的``AppsFlyer``包和谷歌服务；
* 在project的``build.gradle``文件中，添加``mavenCentral``配置，如图：
![image](http://bmob-cdn-20286.b0.upaiyun.com/2018/08/15/f80705f840de5467801c2e11312fe5bf.png)
* 在app模块的``build.gradle``文件中，添加``AppsFlyer``和谷歌服务，所需配置文本为：
```java
implementation 'com.appsflyer:af-android-sdk:4.8.6@aar'
implementation 'com.android.installreferrer:installreferrer:1.0'
implementation 'com.google.android.gms:play-services-auth:15.0.1'

```
如图：
![image](http://bmob-cdn-20286.b0.upaiyun.com/2018/08/15/d2f7f64e4023c3f780618743e9f0517d.png)

注：如果您的``AndroidStudio``版本是在V3.0以下，只需要把``implementation``替换为``compile``即可。

3. ``AndroidManifest.xml``文件配置：
* 权限相关
```java
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<!--使用GooglePay用到-->
<uses-permission android:name="com.android.vending.BILLING" />
```
* 配置``AppsFlyer``所需要的receiver
```java
<receiver android:name="com.appsflyer.SingleInstallBroadcastReceiver" android:exported="true">
    <intent-filter>
        <action android:name="com.android.vending.INSTALL_REFERRER" />
    </intent-filter>
</receiver>
```
## 初始化

在您的项目中的``Application``的``onCreate()``方法中调用``HelperSDK``的``init()``方法，依次传入Application对象和AppFlyers平台申请的开发者Key，用法如下：

```java

public class MyApplication extends Application {

    final String AF_DEV_KEY = "MzXomgBiV4boBSPhZvauZk";
    @Override
    public void onCreate() {
        super.onCreate();
        HelperSDK.init(this,AF_DEV_KEY);
    }
}

```
## 上传用户事件

上传用户行为事件如支付事件，登录事件等，事件数据用一个``Map<String,Object>``封装，调用``HelperSDK``的``reportEvent``方法,依次传入事件类型和代表事件数据的map对象，用法如下：

```java

Map<String,Object> eventMap = new HashMap<>();
eventMap.put("userName","nothing");
eventMap.put("address","guangzhou");
HelperSDK.reportEvent(AFInAppEventType.LOGIN,eventMap);

```

上传后您可以在AppsFlyers控制台上看到事件统计，如下：

![image](http://bmob-cdn-20286.b0.upaiyun.com/2018/08/15/9f76b0a1406e4fc780e674f7f763b329.png)




## Google登录
1. 首先你需要参考[这里](https://developers.google.com/identity/sign-in/android/start-integrating)的``Configure a Google API Console project``步骤，依次创建好应用，上传您应用的包名和对应的shua1签名即可，创建成功如图：
![image](http://bmob-cdn-20286.b0.upaiyun.com/2018/08/15/9722cc3b403c570780f623b2befcd4a4.png)

2. SDK中帮您封装了比较零碎的API调用，也省去了看Google官网文档的时间，用法如下：

* 在需要Google登录的时机如登录按钮的事件监听函数中，调用``HelperSDK``的``googleSign``方法，传入Activity对象，代码如下：
```java
HelperSDK.googleSign(this);
```

* 因为您的应用中需要调起Google服务并在您的``Activity``中接收回调，所以下一步需要在``onActivityResult()``方法中处理，调用``HeplerSDK``的``handleData()``方法，传入``onActivityResult()``方法中的intent数据，您就能获取到一个代表Google账号数据的``GoogleSignInAccount``对象，具体代码如下：

```java

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data){
    super.onActivityResult(requestCode, resultCode, data);
    if (requestCode == HelperSDK.GOOGLE_SIGN_IN) {
        try {
            GoogleSignInAccount account = HelperSDK.handleData(data);

            toast("ok");
            StringBuilder sb = new StringBuilder();
            sb.append("DisplayName: " + account.getDisplayName())
                .append("Account: " + account.getAccount() + "\n")
                .append("Email:" + account.getEmail() + "\n")
                .append("FamilyName: " + account.getFamilyName() + "\n")
                 .append("GivenName: " + account.getGivenName() + "\n")
                 .append("Id: " + account.getId() + "\n")
                 .append("IdToken: " + account.getIdToken() + "\n");
            tv_account_info.setText(sb.toString());
        } catch (ApiException e) {
            e.printStackTrace();
            Log.e(TAG, "signInResult:failed code=" + e.getStatusCode());
            toast("failed");
        }
    }
}

```

注：
1. 关于获取应用的SHA1签名，在``AndroidStudio``中有比较快捷的操作而不用在命令行中繁琐操作，选择右侧的``Gradle``栏并展开App的tasks，如图：

![image](http://bmob-cdn-20286.b0.upaiyun.com/2018/08/15/1fe9051440304c4480de0e694453f247.png)

选中``signingReport``点中后即可看到对应的SHA1值，如图：
![image](http://bmob-cdn-20286.b0.upaiyun.com/2018/08/15/8d0757a2402260dd8038debc2155b25b.png)

2. 第一种方式仅为生成debug的，如果指定自己的keystore，可以在AndroidStudio的Terminal中用命令：

```java
keytool -v -list -keystore ks0821  
```
ks0821为keystore的文件名；

3. 关于签名的坑，当开发者用自己的签名上传应用到Google play时，Google play会去掉开发者的签名并加Google的签名，这会导致谷歌登录失败，具体情况可以参考[这篇文章](https://blog.csdn.net/a940659387/article/details/79527006)的部分，上传后把Google签名的Sha1在提交到Google即可解决登录的问题；

## GooglePay
1. 初始化
> GooglePay利用了IPC去绑定Service，而``HelperSDK``封装了这些细节，在初始化操作中，SDK会绑定Service，你只需要调用``HelperSDK``的``initGoogleBuy()``方法即可，传入当前的Activity对象，建议在``onCreate()``方法中调用，用法如下：
```java
HelperSDK.initGoogleBuy(this);
```
2. 调起购买
> 调用``HelperSDK``的``googleBuy()``方法，依次传入当前Activity对象和商品id，用法如下：
```java
HelperSDK.googleBuy(this,"101");
```
3. 接收回调
> 和登录类似，结果也会回调到当前的Activity，同样需要在``onActivityResult()``方法中调用SDK处理，具体代码如下：
```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if (requestCode == HelperSDK.GOOGLE_BUY) {
        // 这里和Google登录很类似，调用SDK的处理回调数据的方法
        String purchaseData = HelperSDK.handleGoogleBuyData(data);
        // 对purchaseData做空判断
        if (resultCode == RESULT_OK) {
            try {
                JSONObject jo = new JSONObject(purchaseData);
                String sku = jo.getString("productId");
                // token作为消耗商品必传的参数
                String token = jo.getString("purchaseToken");
                toast("You have bought the " + sku + ". Excellent choice,adventurer!");
                
            }
            catch (JSONException e) {
                toast("Failed to parse purchase data.");
                e.printStackTrace();
            }
        }
    }
}
```
4. 释放资源
> 为了避免内存泄露等问题，一般我们都会在``Activity``的``onDestroy()``方法中做释放资源和解绑Service的操作，这里你只需要调用``HelperSDK``的``releaseGoogleBuy()``方法，传入当前的Activity对象即可，SDK帮你做了解绑Service的操作，用法如下：
```java
HelperSDK.releaseGoogleBuy(this);
```
