# 快跑者接口文档

------

- [接口简介](#接口简介)
- [接口说明](#接口说明)
- [账号认证接口](#账号认证接口)
- [获取配送团队信息接口](#获取配送团队信息接口)
- [下单接口](#下单接口)
- [订单状态改变后回调](#订单状态改变后回调)
- [撤销订单接口](#撤销订单接口)
- [查看订单接口](#查看订单接口)
- [附：签名与验签说明](#附签名与验签说明)
    + [请求参数签名](#请求参数签名)
        - [1. 参数筛选](#1-参数筛选)
        - [2. 参数排序](#2-参数排序)
        - [3. 参数拼接](#3-参数拼接)
        - [4. 签名](#4-签名)
    + [返回参数验证签名](#返回参数验证签名)

## 接口简介

本文档系新版快跑者（2017 年版）第三方开放接口文档，任何快跑者商户都拥有快跑者第三方开放接口调用权限！

快跑者商户可通过开放接口向快跑者系统发送配送订单，并可对订单进行撤销、查看等相关操作。

快跑者开放接口使用流程如下：

1. 注册快跑者商户（下载快跑者商户端 APP，使用手机号码注册）；
2. 调用账号认证接口，获取身份认证标识和身份认证密钥；
3. 调用获取配送团队信息接口，获取快跑者商户合作的快跑者配送团队以及配送团队下的配送员和配送群信息；
4. 调用下单接口，向快跑者发送配送订单，配送订单可发往对应商户，也可直接发往团队下的配送员或配送群；
5. 调用查看订单接口，可查看订单的详细信息及订单的状态；
6. [可选]如需撤销订单，可调用撤销订单接口。

## 接口说明

接口的返回数据为 **JSON** 格式，主要由以下三个字段组成：

- **code**：状态码，取值有 200，204；200 表示接口调用成功， 204 表示接口调用失败
- **message**：错误信息，当接口调用失败（状态码为 204）时，返回的错误信息
- **data**：返回数据，接口调用成功（状态码为 200）之后返回的数据

___

**注：**

+ PHP 开发者可直接获取 [Keloop-PHP-SDK 及 Demo](https://github.com/LingdianIT-Com/keloop/blob/master/src/Keloop-PHP-SDK)；
+ .NET、JAVA 开发者请参考支付宝接口中的 [加签验签 SDK](https://doc.open.alipay.com/doc2/detail?treeId=54&articleId=103419&docType=1) 组装数据生成 **sign**；
+ 开发者也可参考 [签名与验签说明](#签名与验签说明) 自由编写处理代码；
+ `access_key` 表示 **身份认证标识** ，`access_sec` 表示 **身份认证密钥**
+ 调用下单接口成功发送配送订单后，系统会返回配送订单单号：`trade_no`，请开发者保存好，用于后续查询和操作订单；

## 账号认证接口

**简要描述：**开发者可根据注册的快跑者商户账号调用本接口获取接口调用的身份认证标识和身份认证密钥。

**请求 URL：** `http://www.keloop.cn/Api/Tp/authorization`

**请求方式：** POST

**请求参数：**

| 参数名 | 必选 | 类型 | 说明 |
|:----:|:---:|:-----:|:-----:|
| `tel` | 是 | `string` | 快跑者商户账号（注册的手机号码） |
| `password` | 是 | `string` | 快跑者商户账号密码 |
| `note` | 否 | `string` | 接口调用者的描述信息，如：三餐美食-兰州拉面，美团外卖-宜宾燃面 |
    
**返回示例**

- 调用成功示例
```
{
    "code": 200,
    "message": "",
    "data": {
        "access_key": "JNLHPRC5",
        "access_sec": "6YAK4WCN"
    }
}
```
- 调用失败示例
```
{
    "code": 204, 
    "message": "快跑者商户密码不正确", 
    "data": [ ]
}
```

**返回参数说明：**

| 返回参数名 | 类型 | 说明 |
|:-----:|:-----:|:-----:|
| `access_key` | `string` | 身份认证标识 |
| `access_sec` | `string` | 身份认证密钥 |

**备注：**开发者调用本接口获取到 `access_key` 和 `access_sec` 之后，请将其保存到数据库，调用其他接口需要使用这两个参数。

## 获取配送团队信息接口

**简要描述：**开发者可调用本接口获取对应快跑者商户合作的快跑者配送团队及配送团队下的配送群和配送员信息。

**请求 URL：** `http://www.keloop.cn/Api/Tp/getTeamMember`

**请求方式：** GET

**请求参数：**

| 参数名 | 必选 | 类型 | 说明 |
|:----:|:---:|:-----:|:-----:|
| `access_key` | 是 | `string` | 身份认证标识，由账号认证接口获取 |
| `sign` | 是 | `string` | 加密密钥，按支付宝密钥排序，可参考**签名与验签说明** |
| `expire_time` | 是 | `int` | 过期时间，时间戳格式，如：1477483702 |

**返回示例**

- 调用成功示例
```
{
    "code": 200, 
    "message": "", 
    "data": {
        "1": {
            "info": {
                "team_id": 1, 
                "team_name": "飓风队"
            }, 
            "courier": [
                {
                    "user_id": 1, 
                    "user_name": "德莱厄斯12"
                }, 
                {
                    "user_id": 2, 
                    "user_name": "长得帅2号"
                }
            ], 
            "group": [
                {
                    "group_id": 3, 
                    "group_name": "西天取经"
                }, 
                {
                    "group_id": 4, 
                    "group_name": "一个群"
                }
            ]
        }, 
        "2": {
            "info": {
                "team_id": 2, 
                "team_name": "千里马"
            }, 
            "courier": [
                {
                    "user_id": 1, 
                    "user_name": "德莱厄斯12"
                }, 
                {
                    "user_id": 2, 
                    "user_name": "长得帅2号"
                }
            ], 
            "group": [
                {
                    "group_id": 1, 
                    "group_name": "一骑当千"
                }
            ]
        }
    }
}
```
- 调用失败示例
```
{
    "code": 204, 
    "message": "账号认证异常", 
    "data": [ ]
}
```

**返回参数说明：**

| 返回参数名 | 类型 | 说明 |
|:-----:|:-----:|:-----:|
| `info` | `object` |  配送团队的信息  |
| `courier` | `array` |  对应配送团队下的所有配送员的信息  |
| `group` | `array` |  对应配送团队下的所有配送群的信息  |
| `team_id` | `int` |  配送团队ID  |
| `team_name` | `string` |  配送团队名称  |
| `user_id` | `int` |  配送团队下的配送员 ID  |
| `user_name` | `string` |  配送团队下的配送员名称  |
| `group_id` | `int` |  配送团队下的配送群 ID  |
| `group_name` | `string` |  配送团队下的配送群名称  |

## 下单接口

**简要描述：**开发者可调用本接口发送配送订单到快跑者系统。

**请求 URL：** `http://www.keloop.cn/Api/Tp/createOrder`

**请求方式：** POST

**请求参数：**

| 参数名 | 必选 | 类型 | 说明 |
|:----:|:---:|:-----:|:-----:|
| `access_key` | 是 | `string` | 身份认证标识，由账号认证接口获取 |
| `sign` | 是 | `string` | 加密密钥，按支付宝密钥排序，可参考**签名与验签说明** |
| `expire_time` | 是 | `int` | 过期时间，时间戳格式，如：1477483702 |
| `order_content` |  否  | `string` |  第三方订单内容，如：1份烧白开(100x1)  |
| `order_note` |  否  | `string` |  第三方订单备注，如：麻辣一点  |
| `order_mark` |  否  | `string` |  第三方订单标识  |
| `order_from` |  否  | `string` |  第三方订单来源 |
| `order_send` |  否  | `string` |  第三方订单送达时间，如：下午六点钟之前送达 |
| `order_no` |  是  | `string` |  第三方订单单号（唯一） |
| `order_time` |  否  | `string` |  第三方订单的下单时间，如：2016-12-31 23:59:59 |
| `order_photo` |  否  | `string` |  第三方订单图片路径  |
| `order_price` |  是  | `float` |  第三方订单货物的总价金额，如：9.99  |
| `platform_name` |  否  | `string` |  第三方订单所属平台名  |
| `customer_name` |  否  | `string` |  第三方订单中接收客户的名字  |
| `customer_sex` |  否  | `string` |  第三方订单中接收客户的性别，如：男，女，未知  |
| `customer_tel` |  否  | `string` |  第三方订单中接收客户的电话  |
| `customer_address` |  否  | `string` |  第三方订单中接收客户的地址  |
| `customer_tag` |  否  | `string` |  第三方订单中接收客户的坐标，如：104.012765,30.714386  |
| `pay_status` |  是  | `int` |  第三方订单的支付方式：0表示已支付、1表示货到付款  |
| `notify_url` |  否  | `string` |  异步通知url，配送订单的状态改变后将调用该回调地址进行通知  |
| `pay_type` |  是  | `int` |  配送订单费用的支付方式，1表示在线支付，2表示事后结算  |
| `pay_fee` |  否  | `float` |  配送订单的配送费用，pay_type 为 1 时必须带上！  |
| `pay_success` |  否  | `int` |  配送订单的配送费用是否支付成功，0 表示未成功，1 表示已成功  |
| `team_id` |  否  | `int` |  配送订单指定的配送团队ID  |
| `group_id` |  否  | `int` |  配送订单指定的配送团队的配送群ID  |
| `courier_id` |  否  | `int` |  配送接受订单的配送员ID  |

**备注：**当没有带 `team_id` 参数时表示将配送单发给商户，由商户在快跑者商户端进行分发，如果带上 `team_id` 参数，则 `group_id` 和 `courier_id` 两个参数必带一个且只能带一个，表示将配送单直接分发给配送群或配送员（如果两个参数都带上了，优先发往配送群）！

**返回示例**

- 调用成功示例
```
{
    "code": 200,
    "message": "",
    "data": {
        "trade_no": "16120709314700002"
    }
}
```
- 调用失败示例
```
{
    "code": 204, 
    "message": "该订单已存在，请勿重复提交", 
    "data": [ ]
}
```

**返回参数说明：**

| 返回参数名 | 类型 | 说明 |
|:-----:|:-----:|:-----:|
| `trade_no` | `string` | 配送订单的订单单号，可使用该值查询订单状态及撤销配送订单 |

## 订单状态改变后回调

**简要描述：**第三方系统将配送单发往快跑者系统之后，配送单被接单、被送达或被撤销时均会向第三方系统回调通知订单改变状态。

**请求 URL：** 本次请求是由快跑者系统向第三方系统发起，请求地址为第三方系统调用发单接口时所传递的回调地址

**请求方式：** POST

**请求参数：**

| 参数名 | 类型 | 说明 |
|:----:|:-----:|:-----:|
| `access_key` | `string` | 身份认证标识，第三方系统调用下单接口时使用的身份认证标识 |
| `sign` | `string` | 加密密钥，按支付宝密钥排序，可参考**签名与验签说明**，主要用于验证请求权限 |
| `expire_time` | `int` | 请求的过期时间，时间戳格式，如：1477483702 |
| `trade_no` | `string` |  状态发生改变的配送订单的订单单号  |
| `state` | `int` |  配送订单状态码，可选值为 4,5,6,7，具体参考下面的订单状态码说明表  |
| `tel` | `string` |  配送员的电话号码  |
| `update_time` | `string` |  配送订单状态的改变时间，如：2017-01-01 01:01:01  |

**订单状态码说明：**

| 状态码 | 说明 | 说明 |
|:----:|:-----:|:-----:|
| `4` | 取单中（已接单/已抢单） | 发往配送群的订单被配送员抢单，或发往配送员的订单被接单 |
| `5` | 送单中（已取单） | 配送员已从取件处领取货物，准备送往收件人 |
| `6` | （已送达） | 配送员已将货物送达到收件人 |
| `7` | （已撤销） | 配送员已撤销订单 |

**注：**快跑者系统会根据第三方系统发单时是否传递回调地址参数来判断是否回调，目前每次订单状态发生改变之后仅回调一次（无论成功与否）。

## 撤销订单接口

**简要描述：**开发者可调用本接口将已发往快跑者的订单撤销（注：只有商户未发单、配送群未抢单、配送员未接单时才可撤销）。

**请求 URL：** `http://www.keloop.cn/Api/Tp/cancelOrder`

**请求方式：** POST

**请求参数：**

| 参数名 | 必选 | 类型 | 说明 |
|:----:|:---:|:-----:|:-----:|
| `access_key` | 是 | `string` | 身份认证标识，由账号认证接口获取 |
| `sign` | 是 | `string` | 加密密钥，按支付宝密钥排序，可参考**签名与验签说明** |
| `expire_time` | 是 | `int` | 过期时间，时间戳格式，如：1477483702 |
| `trade_no` |  是  | `string` |  配送订单的订单单号  |

**返回示例**

- 调用成功示例
```
{
    "code": 200,
    "message": "",
    "data": []
}
```
- 调用失败示例
```
{
    "code": 204, 
    "message": "只有待发单、待抢单和待取单的订单才可被撤销", 
    "data": [ ]
}
```

**返回参数说明：**

略

## 查看订单接口

**简要描述：**开发者可调用本接口查看订单信息及订单状态。

**说明：**为让快跑者开放接口更为简便易用，现在使用直接跳转 URL 的方式来展示订单详情及相关信息，开发者调用下单接口后，快跑者会返回订单的单号（如：16122813523200001），使用该单号可拼接一个 URL 链接（如：`http://www.keloop.cn/show_order/16122813523200001`），直接访问该链接即可查看该订单的详情和状态。

**备注：** 如果需要直接获取订单信息或状态的接口，请联系快跑者官方。

## 附：签名与验签说明

快跑者开放接口的签名与验签方式借鉴了支付宝接口的签名流程：[支付宝签名流程](https://doc.open.alipay.com/doc2/detail.htm?treeId=200&articleId=105351&docType=1)，因此，PHP、JAVA、C、C++、C# 开放者均可借鉴 [支付宝服务端 SDK](https://doc.open.alipay.com/doc2/detail?treeId=54&articleId=103419&docType=1#s4)。

如果开发者不用 SDK，可根据签名规则自己拼写签名方法，具体的签名流程如下：

### 请求参数签名

#### 1. 参数筛选
获取所有请求参数（不包括字节类型参数，如文件、字节流），剔除参数名为 sign、sign_type、key 的参数及参数值为 '' 或 null 的参数。
以获取配送团队接口为例，需要请求的参数有：

```
{
    "sign": "", 
    "expire_time": 1477483702,
    "access_key": "JNLHPRC5"
}
```

经过参数筛选之后参数为：

```
{
    "expire_time": 1477483702,
    "access_key": "JNLHPRC5"
}
```

#### 2. 参数排序
按照参数名第一个字符的键值 ASCII 码递增排序（字母升序排序），如果遇到相同字符则按照第二个字符的键值 ASCII 码递增排序，以此类推。

经过参数排序之后参数为：

```
{
    "access_key": "JNLHPRC5",
    "expire_time": 1477483702
}
```

#### 3. 参数拼接
将排序后的参数与其对应值，组合成“参数=参数值”的格式，并且把这些参数用**&**字符连接起来，此时生成的字符串为待签名字符串，需将待签名字符串和身份认证密钥放入 SHA1 RSA 算法中得出签名（sign）的值。

经过参数拼接之后的字符串为：

```
access_key=JNLHPRC5&expire_time=1477483702
```

#### 4. 签名
使用各自语言对应的 SHA1 With RSA 签名函数（如 PHP 的签名函数是 md5()）利用商户私钥对待签名字符串进行签名。

PHP 的签名方法为：将第三步参数拼接得到的字符串和 access_sec 值拼接起来再进行 MD5 加密，加密后的字符串即为 sign 值。

### 返回参数验证签名

开发者调用下单接口时可以传递一个 notify_url，当快跑者订单的状态发生变化时，会回调该地址，开发者可以对快跑者回调传递的参数进行验证签名，以保证安全。

验证签名，需要将获取到的参数进行签名（参考上面的签名流程），然后再对比 sign 值是否相等，由此进行参数签名验证！