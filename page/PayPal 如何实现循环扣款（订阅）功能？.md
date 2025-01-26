## 引言

在如今的商业环境中，订阅服务已成为一种流行的商业模式，而 PayPal 作为一个广泛使用的支付平台，提供了实现循环扣款的多种接口。本文将详细介绍如何通过 PayPal 实现循环扣款，涵盖不同接口的特点、实现步骤以及注意事项。

## 接口介绍

PayPal 目前提供多种接口方式来实现循环扣款，主要包括：

1. **Braintree**: 由 PayPal 收购的一家公司，除了支持 PayPal 支付外，还提供了全面的信用卡管理和客户信息管理功能。对于使用 Laravel 框架的开发者来说，Braintree 的 **cashier** 解决方案能够无缝集成，便于管理订阅服务。
2. **REST API**: 当前主流的接口方式，适合大多数开发者。若之前使用过 **OAuth 2.0**，理解 REST API 将相对容易。
3. **旧接口（NVP/SOAP API）**: 由于全球范围内都在向 OAuth 2.0 和 REST API 迁移，除非有特殊需求，建议避免使用旧接口。

## 实现步骤

实现循环扣款的过程可以分为以下四个步骤：

### 1. 创建升级计划

创建升级计划对应于 **Plan** 类。在创建时，需要注意以下几点：

- 升级计划创建后默认处于 **CREATED** 状态，需要修改为 **ACTIVE** 状态以便正常使用。
- **Plan** 对象需要设置 **PaymentDefinition** 和 **MerchantPreferences** 两个对象，均不能为空。
- 设置首次扣款费用时，需使用 **setSetupFee** 方法，而 **Agreement** 对象则设置第二次开始的费用。

示例参数如下：

php
$param = [
    "name" => "standard_monthly",
    "display_name" => "Standard Plan",
    "desc" => "Standard Plan for one month",
    "type" => "REGULAR",
    "frequency" => "MONTH",
    "frequency_interval" => 1,
    "cycles" => 0,
    "amount" => 20,
    "currency" => "USD"
];


创建并激活计划的代码如下：

php
public function createPlan($param)
{
    $apiContext = $this->getApiContext();
    $plan = new Plan();

    $plan->setName($param->name)
         ->setDescription($param->desc)
         ->setType('INFINITE'); // 设置为无限循环

    $paymentDefinition = new PaymentDefinition();
    $paymentDefinition->setName($param->name)
                     ->setType($param->type)
                     ->setFrequency($param->frequency)
                     ->setFrequencyInterval((string)$param->frequency_interval)
                     ->setCycles((string)$param->cycles)
                     ->setAmount(new Currency(['value' => $param->amount, 'currency' => $param->currency]));

    $merchantPreferences = new MerchantPreferences();
    $merchantPreferences->setReturnUrl("https://yourwebsite.com/return?success=true")
                       ->setCancelUrl("https://yourwebsite.com/return?success=false")
                       ->setAutoBillAmount("yes")
                       ->setInitialFailAmountAction("CONTINUE")
                       ->setMaxFailAttempts("0")
                       ->setSetupFee(new Currency(['value' => $param->amount, 'currency' => 'USD']));

    $plan->setPaymentDefinitions([$paymentDefinition]);
    $plan->setMerchantPreferences($merchantPreferences);

    // 创建计划并激活
    try {
        $output = $plan->create($apiContext);
        $patch = new Patch();
        $value = new PayPalModel('{"state":"ACTIVE"}');
        $patch->setOp('replace')->setPath('/')->setValue($value);
        $patchRequest = new PatchRequest();
        $patchRequest->addPatch($patch);
        $output->update($patchRequest, $apiContext);
        return $output;
    } catch (Exception $ex) {
        return false;
    }
}


### 2. 创建订阅（Agreement）

创建 **Agreement** 以让用户订阅，注意以下几点：

- 使用 **setStartDate** 方法设置第二次扣款的时间，格式为 **ISO8601**。
- 通过 **getApprovalLink** 方法得到的 URL 中的 **token** 是唯一的，用于识别用户。

示例参数如下：

php
$param = [
    'id' => 'P-26T36113JT475352643KGIHY', // 上一步创建的 Plan ID
    'name' => 'Standard',
    'desc' => 'Standard Plan for one month'
];


代码如下：

php
public function createPayment($param)
{
    $apiContext = $this->getApiContext();
    $agreement = new Agreement();

    $agreement->setName($param['name'])
              ->setDescription($param['desc'])
              ->setStartDate(Carbon::now()->addMonths(1)->toIso8601String());

    $plan = new Plan();
    $plan->setId($param['id']);
    $agreement->setPlan($plan);

    $payer = new Payer();
    $payer->setPaymentMethod('paypal');
    $agreement->setPayer($payer);

    try {
        $agreement = $agreement->create($apiContext);
        return $agreement->getApprovalLink(); // 跳转到 PayPal 网站，等待用户同意
    } catch (Exception $ex) {
        return "创建支付失败，请重试或联系商家。";
    }
}


### 3. 用户同意后，执行订阅

用户同意后，需执行 **Agreement** 的 **execute** 方法以完成订阅。

php
public function onPay($request)
{
    $apiContext = $this->getApiContext();
    if ($request->has('success') && $request->success == 'true') {
        $token = $request->token;
        $agreement = new \PayPal\Api\Agreement();
        try {
            $agreement->execute($token, $apiContext);
            return $agreement;
        } catch (\Exception $e) {
            return null;
        }
    }
    return null;
}


### 4. 获取交易记录

订阅后，交易扣费记录可能不会立即生成。如为空则过几分钟再次尝试。

php
public function transactions($id)
{
    $apiContext = $this->getApiContext();
    $params = [
        'start_date' => date('Y-m-d', strtotime('-15 years')),
        'end_date' => date('Y-m-d', strtotime('+5 days'))
    ];
    try {
        $result = Agreement::searchTransactions($id, $params, $apiContext);
        return $result->getAgreementTransactionList();
    } catch (\Exception $e) {
        Log::error("获取交易记录失败：" . $e->getMessage());
        return null;
    }
}


## 注意事项

在实现 PayPal 循环扣款的过程中，开发者需要注意以下几点：

1. 国内使用 **Sandbox** 测试时可能会遇到连接速度慢的问题，需考虑用户在执行过程中关闭页面的情况。
2. 一定要实现 **webhook**，以便在用户在 PayPal 取消订阅时，网站能够收到通知。
3. 一旦产生 **Agreement**（订阅），除非主动取消，否则将一直生效。在用户切换升级计划时，需确保前一个计划被取消。

## 总结与建议

通过上述步骤，开发者可以有效地集成 PayPal 的循环扣款功能。建议在实际应用中多参考官方文档，并结合具体项目需求进行适当调整。实现过程中，确保用户体验流畅，并处理好各种可能出现的异常情况。

👉 [野卡 | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)