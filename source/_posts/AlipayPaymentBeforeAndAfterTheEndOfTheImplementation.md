---
title: 支付宝支付前后端实现(Vue+Spring Boot)
excerpt: 如题目
categories:
- 技术文章
tags:
- vue
- 轮子
---

## 前言
最近项目需要使用支付宝支付功能，整理一下思路。

## 应用创建与配置
1. 登录支付宝开放平台创建应用； 并视情况需要添加“手机网站支付”和“电脑网站支付”能力；
    手机网站支付可通过浏览器打开支付宝应用程序进行付款，但在微信公众号打开的页面无法通过支付宝进行支付，电脑端的可通过二维码或者登录支付宝账号付款。
2. 密钥配置
    包括应用私钥、应用公钥和支付宝公钥。

    可通过支付宝开放平台开发助手生成应用私钥与应用公钥，然后在开放平台中将应用公钥配置到应用中，生成支付宝公钥。最终程序中需要使用到应用私钥与支付宝公钥，其中应用私钥用于对发往支付宝的消息进行加密，而支付宝公钥用于对支付宝返回的数据进行验签。

具体配置过程如下：
    1. 使用支付宝开放平台助手生成应用私钥与公钥切记生成后将私钥与公钥另外保存；
    2. 登录开放平台配置应用公钥
    3. 点击接口加签方式中的设置按钮，在弹出窗口中将应用公钥复制进去，然后生成支付宝公钥，将公钥复制下来保存，后续在代码中会需要使用到。生成支付宝公钥后应用公钥基本就用不着了，后续的代码主要使用的是应用私钥和支付宝公钥。

## 处理流程概述
支付处理步骤如下所示：

![](https://api2.mubu.com/v3/document_image/a3915ace-dea7-403f-9a74-ae2a0df62612-3807603.jpg)


## 具体实现
### 前端发起
具体支付界面视业务场景而定，用户选择或填写相关信息后，点击确定按钮时调用后端接口生成订单编号，然后将相关信息展示给用户确认，用户确认后再发起付款流程。发起付款流程前端关键代码如下所示：
```javascript
this.$post("/pay/refill", {
    orderNo: this.orderNo,
    fee: this.selectedPlan.refillMoney,
    channel: this.refillChannel === "wx" ? 2 : 1,
    tbAmount: this.selectedPlan.payMoney,
    tbSentAmount: this.isVip ? this.selectedPlan.payMoney * 0.5 : 0,
}).then((resp) => {
    this.refillCompleted = true;

    if (this.refillChannel === "wx") {
        this.$goPath("/user/pay/wx-pay", resp);
    } else {
        // 支付宝
        // 打开单独的支付宝页面
        this.$goPath("/user/pay/ali-pay", resp);
    }
});
```

注意demo项目支持微信与支付宝两种支付方式，因此前端选择完后会将所选择的方式、金额等信息传给后台生成支付表单。

### 支付表单生成
1. 依赖包下载：后端需要使用到阿里的包，maven地址：
    ```java
    <dependency>
        <groupId>com.alipay.sdk</groupId>
        <artifactId>alipay-sdk-java</artifactId>
        <version>4.10.111.ALL</version>
    </dependency>
    ```
2. 订单生成
    前端发起订单生成请求后，后端生成订单记录，并根据微信还是支付宝来判断调用微信还是支付宝接口完成支付表单生成。
    ```java
    /**
    * 充值
    *
    * @param refill 充值
    * @return 支付相关信息
    */
    @PreAuthorize("isAuthenticated()")
    @PostMapping("/refill")
    public PayDTO refill(@RequestBody @Validated RefillDTO refill) {
        UserDTO user = this.getLoginUserOrThrow();
    
        if (null == tradeService.findByNo(refill.getOrderNo())) {
            // 保存订单
            TradeDTO tradeDTO = new TradeDTO();
            ...
                tradeService.save(user, tradeDTO);
        }
    
        // 获取支付表单信息
        if (refill.getChannel().equals(TradeChannel.WX)) {
            // 微信
            String openId = user.getWxOpenId();
            if (refill.getPlatform().equals(1)) {
                openId = user.getMpOpenId();
            }
            return wxPayService.getPayUrl(openId, refill.getOrderNo(), refill.getFee(), TradeType.REFILL, refill.getPlatform());
        } else if (refill.getChannel().equals(TradeChannel.ALIPAY)) {
            // 支付宝
            return aliPayService.getPayForm(refill.getOrderNo(), refill.getFee(), TradeType.REFILL, refill.getPlatform());
        } else {
            throw new UnsupportedOperationException("不支持的支付渠道");
        }
    }
    ```
3. 支付表单生成
    ```java
    /**
    * 获取支付表单
    *
    * @param orderNo   订单号
    * @param fee       订单金额
    * @param tradeType 交易类型
    * @param platform  平台，0：WEB；1：移动端
    * @return 下单表单
    */
    public PayDTO getPayForm(String orderNo, Double fee, TradeType tradeType, Integer platform) {
        boolean isH5 = Optional.ofNullable(platform).orElse(0).equals(1);
    
        AlipayClient alipayClient = new DefaultAlipayClient(API_URL, APP_ID, APP_KEY, "json", "utf-8", ZFB_PUBLIC_KEY, SIGN_TYPE);
    
        Map<String, String> params = MapEnhancer.<String, String>create()
            .put("out_trade_no", orderNo)
            .put("total_amount", String.format("%.2f", fee))
            .put("subject", tradeType.getName())
            .put("product_code", "FAST_INSTANT_TRADE_PAY")
            .get();
    
        String body;
    
        try {
            if (isH5) {
                if (logger.isDebugEnabled()) {
                    logger.debug("return url: {}", h5ReturnUrl);
                }
    
                AlipayTradeWapPayRequest request = new AlipayTradeWapPayRequest();
                request.setBizContent(JSONObject.toJSONString(params));
                request.setReturnUrl(h5ReturnUrl);
                request.setNotifyUrl(notifyUrl);
    
                body = alipayClient.pageExecute(request).getBody();
            } else {
                AlipayTradePagePayRequest request = new AlipayTradePagePayRequest();
                request.setBizContent(JSONObject.toJSONString(params));
                request.setReturnUrl(returnUrl);
                request.setNotifyUrl(notifyUrl);
                body = alipayClient.pageExecute(request).getBody();
            }
        } catch (AlipayApiException e) {
            logger.error("生成支付宝支付表单失败", e);
            throw BusinessException.create("生成支付宝支付表单失败");
        }
    
        if (logger.isDebugEnabled()) {
            logger.debug("支付表单内容: {}", body);
        }
    
        PayDTO payDTO = new PayDTO();
        payDTO.setFee(fee);
        payDTO.setOrderNo(orderNo);
        payDTO.setCodeUrl(body);
        return payDTO;
    }
    ```

其中API_URL为支付宝网关地址（ https://openapi.alipay.com/gateway.do ），APP_ID为支付宝开放平台中的应用id， APP_KEY为前面密钥配置中的应用私钥；ZFB_PUBLIC_KEY为密钥配置中的支付宝公钥。

注意需要根据h5及pc网站类型来判断使用哪个API；API关键参数如下：

![](https://api2.mubu.com/v3/document_image/fe59435a-1dfe-43e9-ac1a-7af448b45778-3807603.jpg)

调用接口成功后将请求返回的数据与订单号、交易金额一起返回给前端，由前端进行下一步处理。

另外在调用接口时，还需要指定returnUrl与notifyUrl，其中returnUrl是支付宝在用户支付完成后跳转到的页面；notifyUrl是用于接收支付宝支付结果的地址（一般为后端地址，由支付宝调用）；

### 前端跳转处理
前端接收到后端生成的支付表单后，跳转到一个专门的页面，将返回的内容加载到页面中，并自动触发表单提交，处理如下：(ali-pay.vue)
```html

<template>
    <!-- 支付宝支付界面 -->
    <div class="ali-pay box-shadow mt-4 border-radius">
        <div v-html="payInfo.codeUrl" ref="pay"></div>
    </div>
</template>
<script>
export default {
    components: {},
    props: [],
    data() {
        return {
            payInfo: {},
        };
    },
    mounted() {
        this.payInfo = this.$route.query;
 
        this.$nextTick(() => {
            this.$refs.pay.children[0].submit();
        });
    },
    methods: {},
};
</script>
<style lang="scss">
.ali-pay {
}
</style>
```
经过以上处理，h5端将会调起支付宝软件进行支付，web端则会显示二维码或账号输入页面。

### 支付返回页面处理及支付状态更新
用户支付完成后，支付宝将会返回生成表单时指定的returnUrl地址，同时会附带订单号在查询参数中；在这个页面中我们可以根据订单号，通过后台调用支付宝接口查询订单状态，然后依此来更新系统订单支付状态。

前端页面如下（待完善，可以在trade/status接口返回查询的订单状态，在页面做相应提示）：
```html
<template>
    <!-- 支付宝支付界面 -->
    <div class="ali-pay-success box-shadow mt-4 border-radius p-4">
        <div>
            支付成功，您可以进入
            <el-button
                type="text"
                @click="$goPath('/user/account/refill-history')"
                >充值记录</el-button
            >查看历史记录
        </div>
    </div>
</template>
<script>
export default {
    components: {},
    props: [],
    data() {
        return {
            payInfo: {},
        };
    },
    mounted() {
        this.payInfo = this.$route.query;
 
        // 更新支付状态
        this.$get("/trade/status", { no: this.payInfo.out_trade_no });
    },
    methods: {},
};
</script><style lang="scss">
.ali-pay-success {
}
</style>
```

后端/trade/status接口实现如下：
```java
    /**
     * 查询交易状态
     *
     * @param no 交易编号
     * @return 交易状态，0：失败，1：成功，2：未知
     */
@GetMapping("/status")
public Integer getTradeStatus(@RequestParam("no") String no) {
    TradeDTO trade = tradeService.findByNo(no);
    if (null == trade) {
        throw BusinessException.create("订单不存在");
    }
 
    if (trade.getState() == 1 || trade.getState() == 0) {
        return trade.getState();
    }
 
    // 交易状态未知时，通过相关支付服务查询状态
    if (trade.getChannel().equals(TradeChannel.WX)) {
        return wxPayService.getTradeStatus(trade);
    } else if (trade.getChannel().equals(TradeChannel.ALIPAY)) {
        return aliPayService.getTradeStatus(trade);
    }
 
    return trade.getState();
}
```

这里先在系统里面查询订单状态（状态可能已经变更，因为我们同时会接收支付宝支付成功的通知），如果订单状态未知，那么通过支付宝或微信支付的接口更新订单状态。

支付宝支付状态查询接口实现：
```java
    /**
     * 获取订单状态
     *
     * @param trade 订单信息
     * @return 订单状态
     */
    public Integer getTradeStatus(TradeDTO trade) {
        AlipayClient alipayClient = new DefaultAlipayClient(API_URL, APP_ID, APP_KEY, "json", "utf-8", ZFB_PUBLIC_KEY, SIGN_TYPE);
 
        AlipayTradeQueryRequest request = new AlipayTradeQueryRequest();
        Map<String, String> params = MapEnhancer.<String, String>create()
                .put("out_trade_no", trade.getNo())
                .get();
        request.setBizContent(JSONObject.toJSONString(params));
        AlipayTradeQueryResponse response;
        try {
            response = alipayClient.execute(request);
        } catch (AlipayApiException e) {
            logger.error("查询订单状态失败", e);
            return trade.getState();
        }
 
        if (logger.isDebugEnabled()) {
            logger.debug("订单信息: {}", JSONObject.toJSONString(response));
        }
 
        if (response.isSuccess()) {
            String tradeStatus = response.getTradeStatus();
            if (tradeStatus.equals("TRADE_SUCCESS") || tradeStatus.equals("TRADE_FINISHED")) {
                tradeService.tradeSuccess(trade);
            } else if (tradeStatus.equals("TRADE_CLOSED")) {
                tradeService.tradeFailed(trade);
            }
        }
 
        return trade.getState();
    }
```

注意查询到结果后，需要更新交易状态，并视情况进行下一步的业务处理，如用户充值的就得增加用户余额等。

### 支付结果接收
至此支付过程基本已经完成。但某些情况用户支付完成后并未返回我们指定的页面，如用户可能直接关闭浏览器等极端情况，因此必须与支付宝推送结果的方式结合，来防止状态不一致。

接收支付宝支付结果的接口实现如下：
```java
@RequestMapping("ali-notify")
public String aliNotify(@RequestParam Map<String, String> params) {
    if (logger.isDebugEnabled()) {
        logger.debug("收到支付宝通知: {}", params);
    }
 
    return aliPayService.parseAndSaveTradeResult(params);
}
```

具体service方法实现如下：
```java
/**
* 解析通知结果并保存
*
* @param params 通知结果
*/
public String parseAndSaveTradeResult(Map<String, String> params) {
    if (logger.isDebugEnabled()) {
        logger.debug("接收到支付宝支付结果: {}", params);
    }
 
    // 结果验签
    boolean signVerified;
    try {
        signVerified = AlipaySignature.rsaCheckV1(params, ZFB_PUBLIC_KEY, "utf-8", SIGN_TYPE);
    } catch (AlipayApiException e) {
        logger.error("验签失败", e);
        return "failure";
    }
    if (!signVerified) {
        logger.warn("验签失败，通知内容：{}", params);
        return "failure";
    }
 
    String orderNo = MapUtils.getString(params, "out_trade_no", "");
    if (StringUtils.isBlank(orderNo)) {
        logger.warn("订单号为空，通知内容：{}", params);
        return "success";
    }
 
    // 根据订单号查询订单
    TradeDTO trade = tradeService.findByNo(orderNo);
    if (null == trade) {
        logger.warn("订单不存在，通知内容：{}", params);
        return "success";
    }
 
    if (trade.getState() == 1) {
        logger.debug("订单已成功");
        return "success";
    }
 
    String tradeStatus = MapUtils.getString(params, "trade_status", "");
    if (tradeStatus.equals("TRADE_SUCCESS") || tradeStatus.equals("TRADE_FINISHED")) {
        trade.setState(1);
        tradeService.tradeSuccess(trade);
    } else {
        trade.setState(0);
        tradeService.tradeFailed(trade);
    }
 
    return "success";
}
```

注意如果查询到支付状态是成功的，就需要进行下一步的业务处理了，如增加用户余额等。

实际交易成功处理，因为同一笔订单可能会有两个进程更新交易状态（通道消息和主动查询的消息），这两个操作肯定只能有一个操作能够生效；如果是单节点情况，可以通过JVM锁来控制；如果多节点的，就需要通过分页式锁来控制了。demo项目里面是采用了Redis实现了个简单的分布式锁处理，具体实现方案网上一大堆，不在此展开。

另外，可以看到demo里面有很多关于微信支付与支付宝支付的判断，这是因为demo同时支持了微信支付与支付宝支付；这两者的支付过程基本类似，本文主要讲的是支付宝支付，微信的未展开，后续再单独对微信支付实现做一个补充。

## 参考链接
[支付宝支付前后端实现(Vue+Spring Boot)](https://blog.csdn.net/leehomlian/article/details/114588616)





