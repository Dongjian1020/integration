# 银联支付：
## 一. 绑定前提:
与银联签约，获取商户号相关的参数。

## 二. 集成步骤：
1. 拷贝相关的依赖文件和jar包到工程目录下，如图：
![image](http://img2016.itdadao.com/d/file/tech/2016/09/15/cd373311150345234.png)
2. 在工程的AndroidManifest.xml文件中注册支付插件使用的Activity和权限 ：

```xml
<!--工程其它配置此处省略…-->
<uses-library android:name="org.simalliance.openmobileapi" android:required="false"/>
<activity
    android:name="com.unionpay.uppay.PayActivity"
    android:label="@string/app_name"
    android:screenOrientation="portrait"
    android:configChanges="orientation|keyboardHidden"
    android:excludeFromRecents="true"
    android:windowSoftInputMode="adjustResize"/>

 <activity
    android:name="com.unionpay.UPPayWapActivity"
    android:configChanges="orientation|keyboardHidden"
    android:screenOrientation="portrait"
    android:windowSoftInputMode="adjustResize"/>

```

```
<uses-permissionandroid:name="android.permission.INTERNET"/>
<uses-permissionandroid:name="android.permission.ACCESS_NETWOR_STATE"/>
<uses-permissionandroid:name="android.permission.CHANGE_NETWORK_STATE"/>
<uses-permissionandroid:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permissionandroid:name="android.permission.READ_PHONE_STATE"/>
<uses-permissionandroid:name="android.permission.ACCESS_WIFI_STATE"/>
<uses-permission android:name="android.permission.NFC" />
<uses-feature android:name="android.hardware.nfc.hce"/>
<uses-permissionandroid:name="android.permission.RECORD_AUDIO"/>
<uses-permissionandroid:name="android.permission.MODIFY_AUDIO_SETTINGS"/>
<uses-permissionandroid:name="org.simalliance.openmobileapi.SMARTCARD" />

```
3. 调用支付接口：

```java
// “00” – 银联正式环境
// “01” – 银联测试环境，该环境中不发生真实交易
String serverMode = "01";
UPPayAssistEx.startPay (activity, null, null, tn, serverMode);
```
4. 支付结果回调：

```
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (data =http://blog.csdn.net/jiangtea/article/details/= null) {
            return;
        }

        String str = data.getExtras().getString("pay_result");
        if (str.equalsIgnoreCase(R_SUCCESS)) {
            // 支付成功后，extra中如果存在result_data，取出校验
// result_data结构见c）result_data参数说明
            if (data.hasExtra("result_data")) {
                String sign = data.getExtras().getString("result_data");
// 验签证书同后台验签证书
// 此处的verify，商户需送去商户后台做验签 
                if (verify(sign)) {
                    //验证通过后，显示支付结果
                    showResultDialog(" 支付成功！ ");
                } else {
// 验证不通过后的处理
// 建议通过商户后台查询支付结果
                }
            } else {
// 未收到签名信息
// 建议通过商户后台查询支付结果
            }
        } else if (str.equalsIgnoreCase(R_FAIL)) {
            showResultDialog(" 支付失败！ ");
        } else if (str.equalsIgnoreCase(R_CANCEL)) {
            showResultDialog(" 你已取消了本次订单的支付！ ");

        }
    }
```
==相关注意事项：==
- 如果需要在APP中验签，则需要自行实现验签公钥更新的机制，否则银联更新密钥后会验签失败。
- [银联开放平台](https://open.unionpay.com/ajweb/help/file)



