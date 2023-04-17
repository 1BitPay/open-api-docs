---
title: API Reference

toc_footers:
  - <div id="language-selector">
    <a href="index.html">English</a> |
    <a href="index_zh.html">简体中文</a>
   </div>
includes:
  - errors

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the 1BitPay API
---

# 简介

欢迎使用1BitPay API，您可以使用我们的API管理交易，自动化签名，管理资产等等，更多的功能马上到来。

# Quick Start

## 创建API Key

请登录商户后台, 左侧导航打开 "ApiKey"，点击新增，创建API Key ，API Key创建成功后，可以使用API Key访问 1BitPay API。

## IP白名单

如果创建 API Key 时设置了白名单，则在调用 1BitPay API 时，只允许从您设置的 IP 白名单地址发起请求。


## 鉴权规则 
- 把公共参数与业务参数合并，去除Sign参数，以及空的参数。
- 把上一步中参数集合的Key按照ASCII排序以后用“=”连接到一块以后。
- 把上一步中做好的串后拼接商户的secret参数。
- 把上一步中生成的串做32位md5小写即可生成参数Sign。

#### 签名示例
- 例如业务参数:
```
  {
    "orderNo":"Or12898771811",
    "name":"John Li"
  }
```
- 将业务参数与公共参数合并并去除Sign得到结果如下：
```
  {
    "orderNo":"Or12898771811",
    "name":"John Li",
    "TimeStamp":1566781991111,
    "Nonce":"dnasja1N",
    "MerchantNo":"meraojiasdoa123",
    "Lang":"en",
    "SignType":"1",
    "ApiKey":"asdhuasdaosd"
  }
```
- 将参数的Key按ASCII排序以后用“=”连接得到的结果如下：
```
  ApiKey=asdhuasdaosd&Lang=en&MerchantNo=meraojiasdoa123&name=John Li&Nonce=dnasja1N&orderNo=Or12898771811&SignType=1&TimeStamp=1566781991111
```
- 假设API Secret=merasdasd,将上一步得到的结果后拼接API Secret得到结果如下：
```
  ApiKey=asdhuasdaosd&Lang=en&MerchantNo=meraojiasdoa123&name=John Li&Nonce=dnasja1N&orderNo=Or12898771811&SignType=1&TimeStamp=1566781991111merasdasd
```
- 将上一步得到的结果进行md5即可得到Sign结果如下
```
  md5(ApiKey=asdhuasdaosd&Lang=en&MerchantNo=meraojiasdoa123&name=John Li&Nonce=dnasja1N&orderNo=Or12898771811&SignType=1&TimeStamp=1566781991111merasdasd)
```
- 注意最后一步为32位小写md5

## 基本信息

### Base URL

  https://api.1bitpay.io


### 公共参数
公共请求参数是每个接口都需要使用到的请求参数，每次请求均需要携带这些参数, 才能正常发起请求。公共请求参数的首字母均为`大写`，以此区分于普通接口请求参数，并且公共参数需要放入到Header中。

参数名 | 类型 | 描述
--------- | ----------- | -----------
Nonce | Int  | 随机6位字符或数字组合
TimeStamp | Int| 13位毫秒级时间戳
MerchantNo| String | 商户编号
SignType|Int|采取的签名类型 1:MD5 2:HASH 目前仅支持MD5
Lang|Strng | 语言 en：英文 zh：中文
Sign|String | 签名值，具体规则详见签名规则描述
ApiKey|String|商户API Key

### 公共响应
公共响应参数如下所示，code为状态码，data为业务响应参数，message为响应结果


参数名 | 类型 | 描述
--------- | ----------- | -----------
code | Int  | 200 成功 详见状态描述
message | String| Success
data|Object| 具体根绝业务会展现不同的数据结构，详见具体业务即可



### 沙盒环境 

  https://sandbox.1bitpay.io

# C2C
### 获取平台汇率
#### 请求地址：/api/otc/rate
#### 请求方式
- Method: POST 
- Content-Type: application/json
#### 请求参数：


参数名 | 类型 | 必要性 | 描述
--------- | ----------- |  ----------- | -----------
| cryptoCurrency  | String |Y|交易币种
| legalCurrency   | String |Y|付款币种
true：为移动端用户使用 false：为PC端用户使用
#### 请求示例
```
{
  "cryptoCurrency":"USDT",
  "legalCurrency":"CNY"
}
```


#### 响应参数
data参数如下：

参数名 | 类型 | 描述
--------- | ----------- | -----------
buyRate | Decimal  | 买单最新汇率
sellRate | Decimal| 卖单最新汇率 
#### 响应示例
```
{
  "code": 200,
  "message": "Success",
  "data": {
    "buyRate":"6.47",
    "sellRate":"6.9"
  }
}
```



### 创建订单接口
#### 请求地址：/api/otc/create
#### 请求方式
- Method: POST 
- Content-Type: application/json
#### 请求参数：

参数名 | 类型 | 必要性 | 描述
--------- | ----------- |  ----------- | -----------
| userName        | String                 |Y|用户姓名
| areaCode        | String                |Y|用户手机区号
| phone           | String         |Y|用户手机号码
| syncUrl         | String    |N|同步回调地址
| asyncUrl        | String    |N|异步回调地址
| cryptoAmount    | Decimal                  |Y|购买或者出售数量
| cryptoCurrency  | String                |Y|交易币种
| legalCurrency   | String                 |Y|付款币种
| idCardType      | Int                   |Y| 证件类型 1：身份证 2：护照
| idCard          | String  |Y|证件号码
| kyc             | Int                   |Y|Kyc级别，目前Kyc级别传递2
| merchantOrderNo | String        |Y|商户订单号
| orderType       | Int                   |Y|订单类型 1：买单 2:卖单
| payWay          | Int                 |Y|支付方式1：银行卡 2:支付宝 3:微信 用户卖单目前仅支持1:银行卡 
| bank            | String                |N| 用户收款开户行，卖单必填
| bankAccount     | String |N|用户收款账户，卖单必填
| bankBranch      | String               |N|用户收款开户支行
|h5|Bool|收银台展现类型 true：移动端适用 false：PC端适用
#### 请求示例
```
{
  "userName":"陈先生",
  "areaCode":"+86",
  "phone":"18848820305",
  "syncUrl":"https://example.com",
  "asyncUrl":"https://example.com",
  "cryptoAmount":"10",
  "cryptoCurrency":"USDT",
  "legalCurrency":"CNY",
  "idCardType":1,
  "idCard":"412627288918989891",
  "merchantNo":"mer1c92b0319ef5b794",
  "merchantOrderNo":"1231222112d8",
  "orderType":1,
  "payWay":"1",
  "bank":"建设银行",
  "bankAccount":"6217229282829299111",
  "bankBranch":"建设支行",
  "h5":false
}
```


#### 响应参数
data参数如下：

参数名 | 类型 | 描述
--------- | ----------- | -----------
url | String  | 收银台地址
isNew | Bool| 是否为新订单 
orderNo|String| 订单号
#### 响应示例
```
{
  "code": 200,
  "message": "Success",
  "data": {
      "url": "http://sanbox.1bitpay.io/1bitpay-open-api-h5/otc.html?t=02d457b0ced0576a69282054911585aa&o=335793449653514240&l=zh",
      "isNew": true,
      "orderNo": "335793449653514240"
  }
}
```



### 资金归集
#### 注意：商户目前需每5分钟调用一次该接口进行资金归集，否则将无法交易

#### 请求地址：/api/transaction/assets/collect
#### 请求方式
- Method: POST 
- Content-Type: application/json
#### 请求参数：

参数名 | 类型 | 必要性 | 描述
--------- | ----------- |  ----------- | -----------
| isMain          | Int    |Y|归集的是否是主链币 1：是 0：否
#### 请求示例
```
{
  "isMain":1
}
```


#### 响应参数
暂无data参数

#### 响应示例
```
{
  "code": 200,
  "message": "Success"
}
```






# MPC Co-Signer

## 概述
  MPC 全称为 Security Multi-Party Computation（[安全多方计算](https://zh.wikipedia.org/wiki/安全多方计算)）。通过 MPC 可以做到从生成、使用、存储整个生命周期中，私钥可用不可见。1BitPay 使用 MPC 技术将传统的单私钥变成了分布式的私钥分片，可以有效避免单私钥带来的单点风险，实现团队多人共同管理资金，并支持社交恢复。详细的私钥管理方案可参考 [私钥管理方案](https://support.1bitpay.io/v/zh/mpcwalletcn/key-management-cn).

  MPC Co-Signer 可以允许通过API自动实现交易批准和签署交易，取代使用移动设备手动审批，非常适合频繁交易的场景，实现自动化审批及签名。

## 下载私钥分片

  根据1BitPay[私钥管理方案](https://support.1bitpay.io/v/zh/mpcwalletcn/key-management-cn)，用户创建钱包后，私钥分为三部分，分片1保存在用户设备，分片2备份到iCloud 或 Google Drive，分片3保存在1BitPay SGX可信服务器，当您需要使用 MPC Co-Signer功能时，需要在商户后台 -> ApiKey模块，申请下载私钥分片，APP确认审批后，允许下载加密后的存储在用户设备上的私钥分片1。

  <aside class="warning">
  私钥分片虽然不是完整的私钥，但是也是资产管理重要的一部分，建议专人操作，并部署在安全环境。
  </aside>

## 签名算法

参与签名的参数如下：

参数名 | 类型 | 必要性 | 描述
--------- | ----------- |  ----------- | -----------
| id          | Int    |Y|待签名列表id
| from          | String    |N| 待签名列表fromAddress：转出地址
| to          | String    |N| 待签名列表toAddress：转入地址
| value          | String    |N| 待签名列表amount： 转入金额
| chainId          | String    |B| 待签名列表chainId 链id
| status          | String    |Y｜交易状态 1：通过交易 2:拒绝交易

#### 签名步骤：
- 获取商户MPC私钥，从商户后台下载即可
- 用MPC私钥对参数的JSON结构的字符串进行加密。
- 将上一步中生成的即为签名参数data，同业务数据一并传入到apprvoe接口进行交易签名即可

#### 签名示例：
- 例如id为1的待签名数据signObject的json结构如下:
```
  signObject:{
    "id":1,
    "from":"TGGh3cGp9P21Ebvg9JitjHoeJaKqrg3bRg",
    "to":"TFMQrPdFWuPzFRXb42sxB22ABCVL6xSopV",
    "value":"4.1",
    "chainId":"56",
    "status":1
  }
```
- 将上一步的json结构转json字符串得到signObjectString

```
  signObjectString = toString(signObj)
```
- 此时用私钥文件对signObjectString字符串加密得到data

  注意：
  - 私钥的密码为"1bitpay"+商户的编号
  - 加密类型采用PKCS12
  - 加密时均采用UTF-8编码

``` 
  data = "HUSDISJDSNDJSJDKSDJSIDJISOADIASLJDALSIDJISALDHAUSIDHA\ASDUAKSD|ADSADAdasdYGYASDHASDJASID"
```



## 获得待签名列表 
#### 请求地址：/api/transaction/pending
#### 请求方式
- Method: POST 
- Content-Type: application/json
#### 请求参数：

参数名 | 类型 | 必要性 | 描述
--------- | ----------- |  ----------- | -----------
| pageNum        | Int                 |Y|当前页码
| pageSize        | Int                |Y|每页数量
#### 请求示例
```
{
  "pageNum":1,
  "pageSize":20
}
```


#### 响应参数
data 结构如下：

参数名 | 类型 | 描述
--------- | ----------- | -----------
total | Int  | 总数量
list | List| 待签名list
list 结构如下：

参数名 | 类型 | 描述
--------- | ----------- | -----------
id | Int  | 唯一标识
fromAddress | String| 转入地址
toAddress | String| 转出地址
amount|Decimal|交易金额
chainId|String|链ID

#### 响应示例
```
{
  "code": 200,
  "message": "Success",
  "data": {
     total:25,
     list:[
      {
        "id":1,
        "fromAddress":"TGGh3cGp9P21Ebvg9JitjHoeJaKqrg3bRg",
        "toAddress":"TFMQrPdFWuPzFRXb42sxB22ABCVL6xSopV",
        "amount":"4.1",
        "chainId":"56"
      }
     ]
  }
}
```
 
## 签名 

#### 请求地址：/api/transaction/approve
#### 请求方式
- Method: POST 
- Content-Type: application/json
#### 请求参数：

参数名 | 类型 | 必要性 | 描述
--------- | ----------- |  ----------- | -----------
| id          | Int    |Y|待签名列表id
| from          | String    |N| 待签名列表fromAddress：转出地址
| to          | String    |N| 待签名列表toAddress：转入地址
| value          | String    |N| 待签名列表amount： 转入金额
| chainId          | String    |B| 待签名列表chainId 链id
| status          | String    |Y｜交易状态 1：通过交易 2:拒绝交易
|data|String|Y|交易签名数据
#### 请求示例
```
{
  "id":1,
  "from":"TGGh3cGp9P21Ebvg9JitjHoeJaKqrg3bRg",
  "to":"TFMQrPdFWuPzFRXb42sxB22ABCVL6xSopV",
  "value":"67.2",
  "chainId":"20",
  "status":1,
  "data":"dahsudiasdoaasidoasdaosdasd9as8d9a0s8d90as8d9a0s8d09asduashdkasdjaksdajksdasjdhakjdha"
}
```


#### 响应参数
暂无data参数

#### 响应示例
```
{
  "code": 200,
  "message": "Success"
}
```

# MPC 钱包

MPC SaaS 钱包，马上到来！
