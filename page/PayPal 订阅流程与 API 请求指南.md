## 1. 在 PayPal 后台创建产品及计划

正式环境创建订阅计划的地址为：
[PayPal 订阅计划创建](https://www.paypal.com/merchantapps/appcenter/acceptpayments/subscriptions)

沙盒环境创建订阅计划的地址为：
[PayPal 沙盒订阅计划](https://www.sandbox.paypal.com/billing/plans)

进入上述链接后，您可以创建产品及相应的计划（请注意，这里创建的产品及计划仅适用于正式环境，沙盒数据不会在此处显示）。

根据提供的流程，您可轻松完成产品及计划的创建。

此外，您还可以通过 API 接口创建产品及计划。

## 2. 使用 API 创建产品及计划

API 地址为：[PayPal API 文档](https://developer.paypal.com/api/rest/)

### 1. 生成 Token

首先，在您的 PayPal 账户中获取 `clientId` 和 `Secret`，然后通过以下地址获取 `access_token`：

- 沙盒环境：`https://api.sandbox.paypal.com/v1/oauth2/token`
- 正式环境：`https://api.paypal.com/v1/oauth2/token`

### 2. 创建产品

以下是创建产品的示例请求：

bash
curl -v -X POST https://api-m.sandbox.paypal.com/v1/catalogs/products \
-H "Content-Type: application/json" \
-H "Authorization: Bearer Access-Token" \
-H "PayPal-Request-Id: PRODUCT-18062020-001" \
-d '{
  "name": "视频流媒体服务",
  "description": "视频流媒体服务",
  "type": "SERVICE",
  "category": "SOFTWARE",
  "image_url": "https://example.com/streaming.jpg",
  "home_url": "https://example.com/home"
}'


- `name`: 产品名称
- `description`: 产品说明
- `type`: 产品类型（例如，PHYSICAL、DIGITAL 或 SERVICE）
- `category`: 产品类别
- `image_url`: 产品 logo
- `home_url`: 产品主站地址

### 3. 创建计划

以下是创建计划的示例请求：

bash
curl -v -X POST https://api-m.sandbox.paypal.com/v1/billing/plans \
-H "Content-Type: application/json" \
-H "Authorization: Bearer Access-Token" \
-H "PayPal-Request-Id: PLAN-18062019-001" \
-d '{
  "product_id": "PROD-XXCD1234QWER65782",
  "name": "视频流媒体服务计划",
  "description": "视频流媒体服务基础计划",
  "status": "ACTIVE",
  "billing_cycles": [
    {
      "frequency": {
        "interval_unit": "MONTH",
        "interval_count": 1
      },
      "tenure_type": "REGULAR",
      "sequence": 1,
      "total_cycles": 12,
      "pricing_scheme": {
        "fixed_price": {
          "value": "6",
          "currency_code": "USD"
        }
      }
    }
  ],
  "payment_preferences": {
    "auto_bill_outstanding": true,
    "setup_fee": {
      "value": "6",
      "currency_code": "USD"
    },
    "setup_fee_failure_action": "CONTINUE",
    "payment_failure_threshold": 3
  },
  "taxes": {
    "percentage": "0",
    "inclusive": false
  }
}'


- `product_id`: 产品 ID
- `name`: 计划名称
- `description`: 计划说明
- `status`: 计划状态
- `billing_cycles`: 一系列计费周期的设置

### 4. 创建订阅

以下是创建订阅的示例请求：

bash
curl -v -X POST https://api-m.sandbox.paypal.com/v1/billing/subscriptions \
-H "Content-Type: application/json" \
-H "Authorization: Bearer &lt;Access-Token&gt;" \
-H "PayPal-Request-Id: SUBSCRIPTION-21092019-001" \
-d '{
  "plan_id": "P-5ML4271244454362WXNWU5NQ",
  "start_time": "2022-07-21T00:00:00Z",
  "quantity": "20",
  "shipping_amount": {
    "currency_code": "USD",
    "value": "10.00"
  },
  "application_context": {
    "brand_name": "walmart",
    "locale": "en-US",
    "shipping_preference": "SET_PROVIDED_ADDRESS",
    "user_action": "SUBSCRIBE_NOW",
    "payment_method": {
      "payer_selected": "PAYPAL",
      "payee_preferred": "IMMEDIATE_PAYMENT_REQUIRED"
    },
    "return_url": "https://example.com/returnUrl",
    "cancel_url": "https://example.com/cancelUrl"
  }
}'


- `plan_id`: 计划 ID
- `start_time`: 实际的首次扣款时间
- `quantity`: 订阅中的产品数量
- `shipping_amount`: 交易的货币和金额

### 5. 订阅详情获取

以下是获取订阅详情的示例请求：

bash
curl -v -X GET https://api-m.sandbox.paypal.com/v1/billing/subscriptions/I-BW452GLLEP1G \
-H "Content-Type: application/json" \
-H "Authorization: Bearer Access-Token"


### 6. 配置 Webhook

您可以根据需求配置 Webhook，以监测订阅的状态和变更。

👉 [野卡 | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)

请根据上述步骤进行操作，以确保订阅的顺利创建与管理。