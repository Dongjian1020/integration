# 支付宝支付:
## 一. 绑定前提:
1. 我们自己要和支付宝签约(商户签约)
2. 秘钥配置

## 二. 集成步骤:
1. 拷贝依赖文件和jar包到工程相应目录下，如图：
![image](http://img.blog.csdn.net/20160202112703880)
2. AndroidManifest.xml文件配置，如下：

```xml
      <!-- alipay sdk begin -->
<activity
android:name="com.alipay.sdk.app.H5PayActivity"
android:configChanges="orientation|keyboardHidden|navigation|screenSize"
android:exported="false"
android:screenOrientation="behind"
android:windowSoftInputMode="adjustResize|stateHidden" >
</activity>
<!-- alipay sdk end -->
```
3. 调用支付接口：

```java
 /**
     * call alipay sdk pay. 调用支付宝SDK支付
     */
    public void alipay(String tradeNo, String price, String notify_url) {
        // 订单
        // String orderInfo = getOrderInfo("测试的商品", "该测试商品的详细描述", "0.01");

        String orderInfo = getOrderInfo(tradeNo, title,
                payInfo, price, notify_url);
        // 对订单做RSA 签名
        String sign = sign(orderInfo);
        try {
            // 仅需对sign 做URL编码
            sign = URLEncoder.encode(sign, "UTF-8");
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }

        // 完整的符合支付宝参数规范的订单信息
        final String payInfo = orderInfo + "&sign=\"" + sign + "\"&" + getSignType();

        Runnable payRunnable = new Runnable() {

            @Override
            public void run() {
                // 构造PayTask 对象
                PayTask alipay = new PayTask(PayActivity.this);
                // 调用支付接口，获取支付结果
                String result = alipay.pay(payInfo);

                Message msg = new Message();
                msg.what = SDK_PAY_FLAG;
                msg.obj = result;
                mHandler.sendMessage(msg);
            }
        };

        // 必须异步调用
        ThreadPoolUtils.execute(payRunnable);
    }

    /**
     * create the order info. 创建支付宝订单信息
     */
    public String getOrderInfo(String tradeNo, String subject, String body, String price, String notify_url) {

        // 签约合作者身份ID
        String orderInfo = "partner=" + "\"" + PARTNER + "\"";

        // 签约卖家支付宝账号
        orderInfo += "&seller_id=" + "\"" + SELLER + "\"";

        // 商户网站唯一订单号
        orderInfo += "&out_trade_no=" + "\"" + tradeNo + "\"";

        if (!TextUtils.isEmpty(subject)) {
            // 商品名称
            orderInfo += "&subject=" + "\"" + subject + "\"";
        }

        if (!TextUtils.isEmpty(body)) {
            // 商品详情
            orderInfo += "&body=" + "\"" + body + "\"";
        }

        // 商品金额
        orderInfo += "&total_fee=" + "\"" + (Integer.parseInt(price) / 100f) + "\"";

        // 服务器异步通知页面路径
        orderInfo += "&notify_url=" + "\"" + notify_url + "\"";

        // 服务接口名称， 固定值
        orderInfo += "&service=\"mobile.securitypay.pay\"";

        // 支付类型， 固定值
        orderInfo += "&payment_type=\"1\"";

        // 参数编码， 固定值
        orderInfo += "&_input_charset=\"utf-8\"";

        // 设置未付款交易的超时时间
        // 默认30分钟，一旦超时，该笔交易就会自动被关闭。
        // 取值范围：1m～15d。
        // m-分钟，h-小时，d-天，1c-当天（无论交易何时创建，都在0点关闭）。
        // 该参数数值不接受小数点，如1.5h，可转换为90m。
        orderInfo += "&it_b_pay=\"30m\"";

        // extern_token为经过快登授权获取到的alipay_open_id,带上此参数用户将使用授权的账户进行支付
        // orderInfo += "&extern_token=" + "\"" + extern_token + "\"";

        // 支付宝处理完请求后，当前页面跳转到商户指定页面的路径，可空
        orderInfo += "&return_url=\"m.alipay.com\"";

        // 调用银行卡支付，需配置此参数，参与签名， 固定值 （需要签约《无线银行卡快捷支付》才能使用）
        // orderInfo += "&paymethod=\"expressGateway\"";

        return orderInfo;
    }

    /**
     * 支付宝支付使用 sign the order info. 对订单信息进行签名
     * 
     * @param content
     *            待签名订单信息
     */
    public String sign(String content) {
        return SignUtils.sign(content, RSA_PRIVATE);
    }

    /**
     * 支付宝支付使用 get the sign type we use. 获取签名方式
     */
    public String getSignType() {
        return "sign_type=\"RSA\"";
    }
```
支付宝支付成功的回调(收到的msg.what为SDK_PAY_FLAG)

```
/**
     * 支付宝支付客户端回调成功
     * @param msg
     * @param theLayout
     */
    private static void aliSdkPaySuc(Message msg, PayActivity theLayout) {
        PayResult payResult = new PayResult((String) msg.obj);
        // 支付宝返回此次支付结果及加签，建议对支付宝签名信息拿签约时支付宝提供的公钥做验签
        String resultInfo = payResult.getResult();
        String resultStatus = payResult.getResultStatus();

        // 判断resultStatus 为“9000”则代表支付成功，具体状态码代表含义可参考接口文档
        if (TextUtils.equals(resultStatus, "9000")) {
            /*Toast.makeText(theLayout, "支付成功",
                    Toast.LENGTH_SHORT).show();*/
            theLayout.payCheck();
        } else {
            // 判断resultStatus 为非“9000”则代表可能支付失败
            // “8000”代表支付结果因为支付渠道原因或者系统原因还在等待支付结果确认，最终交易是否成功以服务端异步通知为准（小概率状态）
            if (TextUtils.equals(resultStatus, "8000")) {
                Toast.makeText(theLayout, "支付结果确认中",
                        Toast.LENGTH_SHORT).show();

            } else if (TextUtils.equals(resultStatus, "4000")) {
                // 其他值就可以判断为支付失败，包括用户主动取消支付，或者系统返回的错误
                Toast.makeText(theLayout, "支付失败",
                        Toast.LENGTH_SHORT).show();

            } else if(TextUtils.equals(resultStatus, "6001")){
                theLayout.payCount("5","");
                Toast.makeText(theLayout, "支付取消",
                        Toast.LENGTH_SHORT).show();

            }else if(TextUtils.equals(resultStatus, "6002")){
                Toast.makeText(theLayout, "网络连接出错",
                        Toast.LENGTH_SHORT).show();
            }
        }
        theLayout.clickAliPay = false;
    }
```

==需要注意的地方==：
- 私钥一定配置正确，我看有好多小伙伴会在sign(orderInfo)时获得的sign为null,最终导致做URL编码的时候报空指针，我也碰到过，就是服务器那边给的RSA_PRIVATE有问题。公钥客户端用不着
- 微信支付只需要传递一个prepayid，而支付宝支付需要穿三个参数，一个是订单号，一个是价格，一个是notify_url
- 支付宝支付创建订单以及订单校验的过程与微信支付类似，因为同在一个地方，要控制好相应的逻辑
- 注意一下price的单位，分和元不要搞错了
- [支付宝开放平台链接](https://doc.open.alipay.com/docs/doc.htm?treeId=204&articleId=105051&docType=1)

