## PayPal 订阅集成步骤

PayPal 自动续费功能的实现主要包含以下关键步骤：

1. 创建并激活订阅计划
2. 用户创建订阅并跳转至 PayPal 网站授权
3. 用户授权后返回网站完成订阅
4. 获取用户账单和扣款结果
5. 处理订阅状态变更通知

## 环境配置

使用 Composer 安装 PayPal SDK：

bash
composer require paypal/rest-api-sdk-php


开发测试可以使用 PayPal Sandbox 环境进行调试。

## 订阅计划管理

### 创建订阅计划注意事项

- 每个不同价格的商品需要创建独立的订阅计划
- 支付类型设置：
  - 必须包含 `REGULAR` 类型支付
  - `TRIAL` 类型支付需要配合业务代码实现新用户优惠
- 订阅协议生效时间必须在当前时间 24 小时以后
- 如需立即收取首次费用，可通过 `MerchantPreferences` 的 `setSetupFee` 设置

## 订阅流程实现

### 创建订阅协议

php
$link = $agreement->getApprovalLink();
parse_str(parse_url($link, PHP_URL_QUERY), $params);
$token = $params['token'];


### 订阅管理要点

- 同一用户可创建多个订阅协议
- 实际扣款时间会有延迟，建议提前一天执行扣款操作
- 可通过 webhook 接收支付通知：
  - 设置路径：My Apps -> REST API apps -> WEBHOOKS
  - 支付成功事件：PAYMENT.SALE.COMPLETED
  - 通过 billing_agreement_id 匹配订阅记录
- 可使用 Agreement::searchTransactions 查询交易历史

👉 [野卡 | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)

需要更便捷的支付解决方案？使用邀请码 ACCPAY 注册 野卡，即可享受优质支付服务。