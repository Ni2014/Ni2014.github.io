---
layout:     post
title:      CloudCardSDK使用文档
subtitle:   SDKDoc of CloudCard
date:       2018-06-29
author:     宸笙
catalog: true
tags:
    - SDKDocs
    
---


# SDK使用文档

   本文档介绍了**SDK的相关功能和接口使用，希望初次使用的你能较好地入门和上手，祝你使用愉快！
   
## Pre 快速入门
### SDK集成与导入
   鉴于现在基本是以AndroidStudio为主流的开发环境，以下主要讲在AS中集成和导入SDK的步骤：
   1. 在项目App的build.gradle文件中配置好so文件放置的目录；<br>
   
   ![image](http://bmob-cdn-12948.b0.upaiyun.com/2018/04/09/fd829d514066f03380167174914477a7.png)

   2. 点击同步gradle，会看到左侧多了一个名为jniLibs的文件夹；<br>
   
   ![image](http://bmob-cdn-12948.b0.upaiyun.com/2018/04/09/b3af16d5407a041c80941fb0d2dd909b.png)
   
   3. 将SDK对应的jar包和so文件以及SDK依赖的jar复制到该文件夹下；<br>
   
   4. 此时，你就可以开始在项目中使用CloudCard的相关功能了。<br>
    
   如果你使用Eclipse作为开发环境，你可以在libs文件夹下添加相关的jar包和so文件，在必要的时候右键选择add to build path即可。
   
### 同步OR异步

   SDK提供的接口多数为异步方法，以下几点需要您先了解下以便更好地进行开发：
   1. 方法若为异步调用，需要您开启子线程去调用方法；
   2. 异步方法的回调方法如onSuccess，拿到数据之后建议通过Handler去进行展示；
   3. 同步方法较为简单，直接通过return得到；
   4. 如果您在使用的过程中产生了困惑，欢迎给我们提供反馈，SDK也在持续改进中。
   
   
  
## PartA 云卡相关功能
### 0 重新设置请求域名
自``v1.0.8``起，为了减少每次重复打包，SDK提供了重新设置请求域名的API，用法如下：
```java
CloudCard.resetDomain("https://api.pcidata.cn:30018/api/service");
```
注：
- 该方法要在``绑定``操作前调用；
- 如果是生产环境，还需要进一步修改caModel，比如：
```java
CloudCard.setCaModel(2);
```
### 1 绑定
   调用CloudCard类的getInstance方法，记得补充ApplicationListener中的getClient回调，SDK需要拿到您的应用ID和终端号，用法如下：
  
     CloudCard.getInstance(context, new ApplicationListener() {
        @Override
        public Client getClient() {
            Client client = new Client();
            client.setAppid(Config.APPID);
            client.setTerminal(Config.TERMINAL);
            return client;
        }

        @Override
        public void onSuccess() {
            // ...
        }

        @Override
        public void onWarning(int i) {
            // ...
        }

        @Override
        public void onError(CloudError cloudError) {
            // ...
        }
     });  
        
### 2 初始化
   对于使用cos1.0的开发者，你只需要调用Initialization方法即可，在InitializationCallback中补充对应的UserUnique字段。
    
    MyApplication.getInstance().getCloudCard().Initialization(new InitializationCallback() {
        @Override
        public String getUserUnique() {
            return Config.USERID;
        }
    
        @Override
        public void onSuccess() {
            // ...            
        }
    
        @Override
        public void onError(CloudError cloudError, String s) {
            log(cloudError.name())
        }
    });
    
   对于使用cos2.0的开发者，你还需要调用cosInit方法，具体如下：
   
    MyApplication.getInstance().getCloudCard().cosInit(new InitCosCallback() {
        @Override
        public void onSuccess(String s) {
            log("cos初始化成功");
        }
    
        @Override
        public void onError(CloudError cloudError, String s) {
            log("cos初始化失败" + s);
        }
    
        @Override
        public void onFinished() {
             // ...
        }
    });
   
   其余接口的使用也依赖于初始化接口的成功调用。

   <b>提示：<br>
   
   1. 调用cosInit方法后，SDK会执行判断本地是否有卡数据，校验卡数据是否正确等操作；
   2. 所以关于首次初始化调用cosInit出现onError方法被回调的情况，你可以把CloudError枚举对象打印出来，如果是卡数据不存在(CloudError.ERR_NOT_DATA),属于正常情况；
   3. 后续进行开卡操作后便会把卡数据下载到本地，如果您的App后续重新打开执行conInit方法之后就能回调onSuccess方法了。
   
   

### 3 用户开卡
   开通云卡，SDK客户端会从后端下载好卡数据，调用如下：
    
    // 1 构建一个Card对象，传入你的CardCode和CardType
    Card card = new Card();
    card.setCardCode(Config.CARDCODE);
    card.setCardType(Config.CARDTYPE);
    
    SimpleDateFormat formater = new SimpleDateFormat("yyyyMMddHHmmss");
    Date curDate = new Date(System.currentTimeMillis());
    String time = formater.format(curDate);
    // 2 订单号，时间 + 6位随机数
    String orederNO = time + Utils.getFixLenthString(6);
    // 3 调用开发方法，如果不是第一次使用可以调用没有orderNo参数的重载方法
    MyApplication.getInstance().getCloudCard().applyCard(card, new CardCallback() {
        @Override
        public void onSuccess(Card card) {
            log("卡片申请成功");
            // 此时可适应做下对回调的Card对象的保存
        }
    
        @Override
        public void onProcess(float v, String s) {
            // ...
        }
    
        @Override
        public void onError(CloudError cloudError, String s) {
            log("卡片申请失败 " + cloudError + "：" + s );
        }
    
        @Override
        public void onFinished() {
            // ...
        }
    },orederNO);
 
### 4 卡片充值
   充值云卡，分别传入卡对象，交易信息，回调callback即可，用法如下：
   
    // 1 设置卡信息，假设这里的cityCardInfo是在上一步开卡操作中的onSuccess方法回调的Card对象赋值过来的
    Card card = new Card();
    card.setId(cityCardInfo.getId());
    card.setCardCode(cityCardInfo.getCardCode());
    card.setCardType(cityCardInfo.getCardType());
    
    // 2 构造交易信息参数
    SimpleDateFormat formatter = new SimpleDateFormat("yyyyMMddHHmmss");
    String data = formatter.format(new Date(System.currentTimeMillis())) + "123456";
    // 这里可能容易出错，请仔细校对
    String transInfo = "{transType:\"0201\",transAmount:\"" + amount + "\"+,orderNo:\"" + data + "\"}";
    // 3 调用充值方法
    MyApplication.getInstance().getCloudCard().rechargeCard(card, transInfo, new CardCallback() {
        @Override
        public void onSuccess(Card card) {
            log("充值成功");
        }
    
        @Override
        public void onProcess(float v, String s) {
            // ...
        }
    
        @Override
        public void onError(CloudError cloudError, String s) {
            log("充值失败" + cloudError + ":" + s);
        }
    
        @Override
        public void onFinished() {
            // ...
        }
    });
    
### 5 查询云卡信息
   查询云卡信息，可以查看诸如逻辑卡号，余额等信息，用法如下：
   
    // 1 设置卡片信息
    Card card = new Card();
    card.setCardType(Config.CARDTYPE);
    card.setCardCode(Config.CARDCODE);
    // 2 查询
    TransportationCard cardInfo = MyApplication.getInstance().getCloudCard().getTransportationCard(card);
    log("卡号：" + cardInfo.getLogicId());
    log("余额：" + cardInfo.getBalance());

### 6 查询交易记录
   查询云卡交易记录，包含诸如刷卡，充值，时间，余额等信息，用法如下：
   
    // 参数分别对应卡信息，分页索引，条数
    String result = MyApplication.getInstance().getCloudCard().getOnLineTransactionList(cityCardInfo, 0, 30);

   注意：该方法不能在主线程中直接调用，建议新开线程调用，参见SDKDemo中的例子代码。

### 7 更新卡
   更新云卡，如果本地有未上传的交易记录，会先上传后下载卡数据到本地，不过你只需要知道简单用法即可：
   
    MyApplication.getInstance().getCloudCard().update(new CardCallback() {
        @Override
        public void onSuccess(Card card) {
            log("更新云卡信息成功");
        }
    
        @Override
        public void onProcess(float v, String s) {
            // ...
        }
    
        @Override
        public void onError(CloudError cloudError, String s) {
            log("更新云卡信息失败," + cloudError + ": " + s);
        }
    
        @Override
        public void onFinished() {
            // ...
        }
    }); 
    
### 8 获取卡状态
   获取云卡状态，true表示正常能使用，用法如下：
   
    MyApplication.getInstance().getCloudCard().getCardState();
### 9 清除卡片
   清除本地的卡数据，在切换用户的时候调用，调用后需要重新初始化才能继续调用其他接口，使用如下：
   
    MyApplication.getInstance().getCloudCard().clean(new CardCallback() {
        @Override
        public void onSuccess(Card card) {
            log("清卡成功");
        }
    
        @Override
        public void onProcess(float v, String s) {
            // ...
        }
    
        @Override
        public void onError(CloudError cloudError, String s) {
            log("清卡失败 " + cloudError + ":" + s);
        }
    
        @Override
        public void onFinished() {
            // ...
        }
    });

注意：不要使用成功回调时的card参数，因为此时回调过来的是null。

### 10 NFC相关
   参见SDKDemo中的HceService类，使用前先在manifest文件中添加NFC权限
   
        <uses-permission android:name="android.permission.NFC" />

   当终端机和你的手机交互时，HceService中的processCommandApdu方法会被调用，
    
        @Override
        public byte[] processCommandApdu(byte[] commandApdu, Bundle extras) {
            // 调用SDK的Runtime类去处理指令
            byte[] result = rt.process(commandApdu, this);
            return result;
        }
   
   建议你在onSuccess和onError回调中都去做一下更新操作，详见HceService中的代码。


## PartB 云码相关功能
   基本使用流程为：调用时设置卡的CardCode信息，请求生码，SDK接收到加密后的二维码字符串后解密，解密后回调给开发者，开发者可以用zxing库解析字符串并加载出二维码。<br>
   以下的例子基本是SDK的几个生码接口的使用，至于怎么用zxing库解析二维码字符串可以参见SDKDemo中的相关例子代码。
### 二维码开户
   开通二维码账户，使用如下：
   
    MyApplication.getInstance().getCloudCard().applyQrCode(Config.USERID, Config.CARDTYPE, Config.CARDCODE, new QrcodeOpenCallback() {
        @Override
        public void onSuccess(RespQrCodeOpenAccount respQrCodeOpenAccount) {
            log("开码成功");
            log(respQrCodeOpenAccount.getAccountNo());
            log(respQrCodeOpenAccount.getUid());
        }

        @Override
        public void onError(CloudError cloudError, String s) {
            log("开户失败");
            log(cloudError.toString() + "::" + s);
        }

        @Override
        public void onFinished() {
            // ...
        }
    });    
     
### jiadu二维码
   区别与交通部下的二维码，具体使用如下：
            
    MyApplication.getInstance().getCloudCard().getQrcodeDate(Config.CARDCODE,Config.TELEPHONE, Config.UID, Config.USERID, new QrcodeDataCallback() {
        @Override
        public void onSuccess(QrCodeData qrCodeData) {
            // 这里你可以拿到一些参数 SDKDemo中是跳转到新的页面去展示二维码
            Intent intent = new Intent();
            intent.putExtra("mode","0");
            intent.putExtra("CertUselessTime",qrCodeData.getCertUselessTime());
            intent.putExtra("uid",qrCodeData.getUid());
            intent.putExtra("uiid",qrCodeData.getUiId());
            intent.putExtra("certStartTime",qrCodeData.getCertStartTime());
            intent.setClass(QrcodeActivity.this,QrCodeShowActivity.class);
            startActivity(intent);
        }
    
        @Override
        public void onError(CloudError cloudError, String s) {
            // ...
        }
    
        @Override
        public void onFinished() {
            // ...
        }
     },false);
    
   关于最后一个参数isOldType，是为了区分是否用旧版本的方式，如果你用的是旧版本的话，可以直接调用没有isOldType参数的重载方法。
   
### 在线二维码
   在线生成交通部的二维码，用法如下：
    
    MyApplication.getInstance().getCloudCard().getOnlineQrcodeStr(Config.USERID, Config.TELEPHONE, "2017120110581325", "0009", new QrcodeCallback() {
        @Override
        public void onSuccess(String s) {
            //结果用handle 回调展示
            Intent intent = new Intent();
            intent.putExtra("mode", "1");
            intent.putExtra("qrCodeStr", s);
            intent.setClass(QrcodeActivity.this, QrCodeShowActivity.class);
            startActivity(intent);
        }
    
        @Override
        public void onError(CloudError cloudError, String s) {
            log("生码失败");
            log(s);
        }
    
        @Override
        public void onFinished() {
            // ...
        }
    });
 
### 离线二维码
   离线生成交通部的二维码，用法如下：
   
    MyApplication.getInstance().getCloudCard().getOfflineQrcodeStr(Config.USERID, Config.TELEPHONE, "2017120110581325", Config.CARDCODE, new QrcodeCallback() {
        @Override
        public void onSuccess(String s) {
            //结果用handle 回调展示
            Intent intent = new Intent();
            intent.putExtra("mode", "2");
            intent.putExtra("qrCodeStr", s);
            intent.setClass(QrcodeActivity.this, QrCodeShowActivity.class);
            startActivity(intent);
        }
    
        @Override
        public void onError(CloudError cloudError, String s) {
            log("生码失败");
            log(s);
        }
    
        @Override
        public void onFinished() {
            // ...
        }
    });
    
### 二维码充值
## PartC TTP用户接口相关功能
### pre 
   1. 原有开发者接入TTP平台的方式只能是根据文档自行封装，需要额外的调试等工作量；
   2. SDK封装并整合了这些接口，直接给开发者使用；
   3. 由于有的接口需要的参数太多(比如注册接口)，SDK目前提供了两种风格的接口，一种传常用参数，另一种传全部参数(含可选)，后文会对所有参数做必要的描述，也请您耐心看完，后续SDK会逐步改善既有的接口设计；
   4.使用TTP用户接口功能时，为了方便您的使用，SDK新增了TPPManager类，您在使用下述功能前需要先得到TTPManager对象，方法如下：
   
    ttpManager = TTPManager.getTTPmaneger(context, Config.APPID, Config.TERMINAL, Config.KEY);
### 注册
   注册成为云平台的用户；
   参数描述：
   1. userId(用户号)
   2. userName(用户名称)
   3. password (用户密码)
      SDK需要的是加密后的密文，加密方式为sha1，具体可以参考SDKDemo中的做法
   4. sexText(性别)
   
    |  传值  |   00   |   01   |
    | ------ | ------ | ------ |
    |   含义  |   男   |   女   |
   5. mobileNo (手机号码)
   6. areaCode (国际区间码)
   7. emailNo (用户邮箱)
   8. userType (用户类型)
   
    |    传值     |     00      |     01      |     03      |
    |-------------|-------------|-------------|-------------|
    |    含义      |   个人用户   |  企业商业用户 | 个人商业用户 |
   9. appSeid (App SEID 码)
   10. channelFlag(渠道)
   
    |   传值   |    01    |    02    |    03    |
    |----------|----------|----------|----------|
    |   含义   |    pc    |    app   |    pos   |
    
   用法如下(以传通用参数接口为例，对于传全部参数的接口用法是类似的，下同)：
    
    String userId = user_id_et.getText().toString();
    String userName = username_et.getText().toString();
    String pwd = EncryptUtil.getSha1(pwd_et.getText().toString());
    String mobile = mobile_number_et.getText().toString();
    String userType = user_type_et.getText().toString();
    ttpManager.registerCloud(userId, userName, pwd, mobile, userType, new TTPCallback() {
        @Override
        public void onSuccess(String s) {
            // SDK会回调一个customerId字符串，用来唯一标识平台上的用户
            // 建议保存这个字段，登出操作需要用到
            log("注册成功，customerId是" + s);
        }
    
        @Override
        public void onError(CloudError cloudError, String s) {
            log("注册出错 " + cloudError.name() + s);
        }
    
        @Override
        public void onFinished() {
    
        }
    });

### 判断用户是否存在
   查询输入的用户在云平台是否存在，具体结果看回调方法的参数，用法如下：
   参数描述：
   1. checkType 
      |  传值  |   01    |   02    |   03    |   04    |
      |-------| ------- | ------- | ------- | ------- |
      | 含义   |  用户名  |  手机号  |  邮箱    | 用户号   |
   2. checkValue
      根据checkType的区别传不同的值，参考例子代码。
   
    String phoneNumber = phone_number_et.getText().toString();
    ttpManager.isRegisterCloud("02", phoneNumber, new TTPCallback() {
        @Override
        public void onSuccess(String s) {
            // s为0 表示不存在 其余返回存在个数
            log("查询成功，结果为 " + s);
        }
    
        @Override
        public void onError(CloudError cloudError, String s) {
            log("查询出错 " + cloudError.name() + ":" + s);
        }
    
        @Override
        public void onFinished() {
    
        }
    });

### 登录
   平台提供的用户登录功能；
   参数描述：
   1. userId(用户号)
   2. password(用户密码)
        SDK需要的是加密后的密文，加密方式为sha1，具体可以参考SDKDemo中的做法。
   3. mobileNo(手机号码)
   4. appName(手机设备型号)
   5. channelFlag(渠道)
   
       |   传值   |    01    |    02    |    03    |
       |----------|----------|----------|----------|
       |   含义   |    pc    |    app   |    pos   |
   
   用法如下：
   
    String userName = user_name_et.getText().toString();
    String pwd = EncryptUtil.getSha1(pwd_sha1_et.getText().toString());
    ttpManager.cloudUserLogin(userName, pwd, new TTPCallback() {
        @Override
        public void onSuccess(String s) {
            // 登录成功后会得到一个customerId字符，和前文注册时一样
            log("登录成功");
            customerId = s;
        }
    
        @Override
        public void onError(CloudError cloudError, String s) {
            log("登录失败 " + cloudError.name() + ":" + s);
        }
    
        @Override
        public void onFinished() {
    
        }
    });

### 登出
   登出功能，需要传入注册和登录成功回调的customerId；
   参数描述：
   1. customerId(客户在平台的唯一标识)
   2. quitType(退出类型)
    
       |  传值   |   00   |   01   |
       |---------|--------|--------|
       |  含义   | 正常退出 | 异常退出|
       
   3. channelFlag(渠道)

       |   传值   |    01    |    02    |    03    |
       |----------|----------|----------|----------|
       |   含义   |    pc    |    app   |    pos   |

   
   用法如下：
   
    ttpManager.cloudUserLogout(customerId, new TTPCallback() {
        @Override
        public void onSuccess(String s) {
            // 此时s的内容为"成功退出"，作为开发者，你只需要登出成功即可
            log("登出成功");
        }
    
        @Override
        public void onError(CloudError cloudError, String s) {
            log("登出失败 " + cloudError.name() + ": " + s);
        }
    
        @Override
        public void onFinished() {
    
        }
    });

### 查询
   查询用户信息；
   参数描述：
   1. queryFlag(查询标识)
   2. customerId(平台唯一的用户号)
   3. accountNo
   4. queryType
   5. pageFlag(分页标识)
   6. turnPageBeginPos
   7. turnPageShowNum
   8. channelFlag(渠道)
   
       |   传值   |    01    |    02    |    03    |
       |----------|----------|----------|----------|
       |   含义   |    pc    |    app   |    pos   |
   
   用法如下：
   
    ttpManager.getCloudUserInfoList(Config.USERID, new TTPUserInfoCallback() {
        @Override
        public void onSuccess(RespCloudUserInfoListDetail respCloudUserInfoListDetail) {
            log("查询成功");
            // 可以把RespCloudUserInfoListDetail的各字段打印出来，参考SDKDemo
            MLog.logObj(respCloudUserInfoListDetail);
        }
    
        @Override
        public void onError(CloudError cloudError, String s) {
            log("查询失败 " + cloudError.name() + ":" + s);
        }
    
        @Override
        public void onFinished() {
    
        }
    });
   
### 锁卡和解锁

   业务场景：当云卡需要退余额时需要先锁卡，其他功能暂时关闭使用，退余额成功后再进行解锁操作；
   SDK中统一提供了lockUnlockCloud方法,对于的锁卡和解锁操作分别提供lockCloud和unlockCloud方法；
   下为锁卡和解锁的全部参数描述：
   
   1. customerId(平台唯一的用户号)
   2. language(语言类型)
   
        |   传值   |    01    |    02    |    03    |
        |----------|----------|----------|----------|
        |   含义   |  简体中文 |  繁体中文  |   英文   |
     
   3. lockState(锁卡操作必填)
    
        |  传值   |   0303    |   0306    |   2101    |   2201    |
        |---------| --------- | --------- | --------- | --------- |
        |  含义   |  退卡锁卡  |  锁卡锁码  |  单独锁码    | 单独解码   |
   4. reason(如锁卡原因)
   5. card_id(云卡卡号)
   6. channelFlag(渠道)

       |   传值   |    01    |    02    |    03    |
       |----------|----------|----------|----------|
       |   含义   |    pc    |    app   |    pos   |

   7. qrAccountNo(二维码账号)
        锁云码必传
   
#### 锁卡   
         
   一般是为了后面的退余额才做的操作，用法如下：
   
    ttpManager.lockCloud(Config.USERID, "0303", "想退余额", Config.CARDID, 
    Config.QRCODEACCOUNT,new TTPCallback() {
        @Override
        public void onSuccess(String s) {
            // 此时的s为 "操作成功"
            log("锁卡成功");
        }
    
        @Override
        public void onError(CloudError cloudError, String s) {
            log("锁卡失败" + cloudError.name() + ":" + s);
        }
    
        @Override
        public void onFinished() {
    
        }
    });

#### 解锁

   解锁云卡，用法如下：
   
    ttpManager.unlockCloud(Config.USERID, "0303", "恢复正常啦", Config.CARDID,Config.QRCODEACCOUNT, new TTPCallback() {
        @Override
        public void onSuccess(String s) {
            // 此时的s为 "操作成功"
            log("解锁成功");
        }
    
        @Override
        public void onError(CloudError cloudError, String s) {
            log("解锁失败" + cloudError.name() + ":" + s);
        }
    
        @Override
        public void onFinished() {
    
        }
    });

### 退余额
   云卡云码退余额功能；
   参数描述：
   1. customerId(平台用户号)
   2. actionFlag
   
       |  传值   |         0          |          1         |         2         |         3         |         4         |         5         |
       |--------| ------------------ | ------------------ |------------------ |------------------ |------------------ |------------------ |
       |  含义   |   退基本户加云卡余额  |    仅退云卡余额      |   仅退基本户余额    |    仅退云码户余额   |  退基本户加云码户余额 |     未退余额      |
       
   3. accountNo(云卡或云码帐号)
   4. refundOrderId(商户订单号，不能含“-”或“_”，只能字母和数字，长度为8到32位；建议格式： yyyyMMddhhmmssSSS +六位随数。位数商家定，保证唯一第三方代付，需要第三方的订单号)
   5. refundTxnAmt(退款金额 以分为单位)
   6. payChannel(退款渠道),具体含义如下：
   
       |   传值   |   1001   |    1002   |   1003   |   1008   |
       | -------- | -------- | --------- | -------- | -------- |
       |   含义    |  银联通道 |  微信通道  | 支付宝通道 | 第三方代付|
       
   7. channelFlag
   
       |   传值   |    01    |    02    |    03    |
       |----------|----------|----------|----------|
       |   含义   |    pc    |    app   |    pos   |
   
   8 language(语言类型)
   
       |   传值   |    01    |    02    |    03    |
       |----------|----------|----------|----------|
       |    含义   |  简体中文 |  繁体中文 |   英文   |
   
   
   
   
   用法如下：
   
    String refundMoney = refund_amount_et.getText().toString();
    SimpleDateFormat formatter = new SimpleDateFormat("yyyyMMddhhmmssSSS");
    // 构造参数，后面可加随机数
    String data = formatter.format(new Date(System.currentTimeMillis())) + "123456";//订单号（时间+6位随机数）
    ttpManager.refundCloud(Config.USERID, "1", Config.CARDID, data, refundMoney, "1008", new TTPRefundCallback() {
        @Override
        public void onSuccess(RespCloudRefund respCloudRefund) {
            log("refund success ");
            // 可以把不同字段信息都打印出来     
            MLog.logObj(respCloudRefund);
        }
    
        @Override
        public void onError(CloudError cloudError, String s) {
            log("退余额失败" + cloudError.name() + ":" + s);
        }
    
        @Override
        public void onFinished() {
    
        }
    });

### 退卡查询
   查询退余额，用法如下：

    ttpManager.queryRefundCloud(Config.USERID, new TTPRefundCardCallback() {
        @Override
        public void onSuccess(RespCloudIRefundCardList respCloudIRefundCardList) {
            log("查询成功");
            // 你可以把RespCloudIRefundCardList的字段都打印出来 参考SDKDemo
            MLog.logObj(respCloudIRefundCardList);
        }

        @Override
        public void onError(CloudError cloudError, String s) {
            log("查询出错" + cloudError.name() + ":" + s);
        }

        @Override
        public void onFinished() {

        }
    });

    
## PartD 常见错误
参见CloudError枚举，如下表：
    
    
    
       |   枚举   |    描述    |
       |----------|------------|
       | ERR_NOT_USERID |  没传用户ID| 
       | ERR_TOKEN |  token失效| 
       | ERR_NOT_CARD |  后台返回没卡| 
       | ERR_NOT_ROOT |  不支持Root设备| 
       | ERR_CARD_LOCK |  卡数据被锁| 
       | ERR_CRC |  卡数据CRC校验出错| 
       | ERR_NOT_DATA |  没有卡数据| 
       | ERR_JSON |  Json解析错误| 
       | ERR_THREAD_NUM |  已超过最大线程数| 
       | ERR_LENGTH |  二维码证书长度错误| 
       | ERR_NOT_UPDATE |  未更新| 
       | ERR_NULL |  请求失败或公钥下载失败| 
       | ERR_PROCESS |  错误中断| 
       | ERR_PARAMS |  参数错误| 
       | ERR_PARAMS_EMPTY |  参数为空| 
       | ERR_SECURITY |  导入证书失败|
       | GEN_FAIL |  生成卡数据失败|
       
