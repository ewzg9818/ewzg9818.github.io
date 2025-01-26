## 一、引言
PayPal作为全球广泛使用的支付解决方案，其Checkout和Subscription功能为商家提供了便捷的支付处理方式。本文将详细介绍这两种支付方式的生命周期及其具体实现步骤。

## 二、支付生命周期

### 1. Checkout - 收银台支付
Checkout支付流程如下：
1. 本地应用组装参数并请求Checkout接口，接口将同步返回一个支付URL。
2. 本地应用重定向到该URL，用户登录PayPal账户并确认支付，完成后用户将被跳转至设定的本地应用地址。
3. 本地请求PayPal执行付款接口以发起扣款。
4. PayPal将异步通知发送至本地应用，收到数据包后进行验签操作。
5. 验签成功后，进行支付完成后的业务处理，如修改本地订单状态、增加销量、发送邮件等。

php
// 示例代码：Checkout方法
public function checkout(Order $order) {
    ...
    // 创建支付请求
    ...
}


### 2. Subscription - 订阅支付
Subscription支付流程如下：
1. 创建一个支付计划。
2. 激活该计划。
3. 使用已激活的计划创建一个订阅申请。
4. 本地跳转至订阅申请链接以获取用户授权并完成第一期付款，用户支付后携带token跳转至设定的本地应用地址。
5. 回跳后请求执行订阅。
6. 收到订阅授权的异步回调结果，验证支付异步回调成功后进行支付完成后的业务处理。

php
// 示例代码：创建订阅
public function createAgreement(Plan $param, Order $order) {
    ...
    // 创建订阅请求
    ...
}


## 三、具体实现

### 1. 安装与配置
首先，安装PayPal SDK并创建配置文件：
bash
$ composer require paypal/rest-api-sdk-php:* // 使用最新版本
$ touch config/paypal.php

配置内容如下：
php
<?php
return [
    'sandbox' => [
        ...
    ],
    'live' => [
        ...
    ],
];


### 2. PayPal服务类的创建
创建一个`PayPalService`类以封装PayPal的API调用：
php
namespace App\Services;

class PayPalService {
    ...
    public function checkout(Order $order) {
        ...
    }
}


### 3. 支付控制器
创建`PaymentsController`和`SubscriptionsController`以处理支付逻辑和回调：
php
class PaymentsController extends Controller {
    public function payByPayPalCheckout(Order $order) {
        ...
    }
}


### 4. Webhook事件的设置
在PayPal开发者中心创建Webhook事件，确保输入正确的回调地址，并选择相关事件类型。

### 5. 测试与调试
使用PayPal的沙箱环境进行测试，确保所有支付流程正常工作。

## 四、总结
通过本文，我们详细解析了PayPal Checkout和Subscription的支付流程及其实现方法。希望这些信息能够帮助开发者更好地集成PayPal支付功能，提升用户体验。

👉 [野卡 | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)