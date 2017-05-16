# 微信支付：
## 一. 绑定前提:
在微信开放平台注册app获取相关参数。

## 二. 集成步骤：
1. 拷贝相关的依赖文件和jar包到工程目录下，如图：
![image](http://img.blog.csdn.net/20160202105552636)
2. 调用支付接口：

```java
/**
     * 微信支付使用
     * @return
     */
    private String genNonceStr() {
        Random random = new Random();
        return MD5.getMessageDigest(String.valueOf(random.nextInt(10000)).getBytes());
    }

    /**
     * 微信安装检测
     * @return
     */
    private boolean checkApkExist() {
        if (msgApi.isWXAppInstalled()) {
            return true;
        } else {
            return false;
        }
    }

    /**
     * 微信支付：调用微信支付
     * @param prepayId 支付id
     */
    private void weiXinPay(String prepayId) {
        PayReq req = new PayReq();

        req.appId = APP_ID;
        req.partnerId = MCH_ID;
        req.prepayId = prepayId;
        req.packageValue = "Sign=WXPay";
        req.nonceStr = genNonceStr();
        req.timeStamp = String.valueOf(System.currentTimeMillis() / 1000);

        StringBuilder sb = new StringBuilder();
        sb.append("appid=");
        sb.append(req.appId);
        sb.append("&noncestr=");
        sb.append(req.nonceStr);
        sb.append("&package=");
        sb.append(req.packageValue);
        sb.append("&partnerid=");
        sb.append(req.partnerId);
        sb.append("&prepayid=");
        sb.append(req.prepayId);
        sb.append("&timestamp=");
        sb.append(req.timeStamp);
        sb.append("&key=");
        sb.append(API_KEY);

        String appSign = MD5.getMessageDigest(sb.toString().getBytes()).toUpperCase();

        req.sign = appSign;
        msgApi.registerApp(APP_ID);
        msgApi.sendReq(req);
    }
```
==相关注意事项：== 
- AndroidManifest.xml中package名字和项目包名一样；
- 将WXPayEntryActivity.java放在package.wxapi/下面
- AndroidManifest.xml中添加.wxapi.WXPayEntryActivity（不添加，支付成功后无法跳转到相应的通知Activity界面）；
- 应用需要签名，并且和平台所提交应用签名一致，[签名核对工具下载](https://open.weixin.qq.com/zh_CN/htmledition/res/dev/download/sdk/Gen_Signature_Android.apk)。
- [微信开放平台连接](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=8_1)
