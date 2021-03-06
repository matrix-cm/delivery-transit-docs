# API对接文档

> 该文档为初稿设计, 后续接口数据格式可能会有调整.

## 一. 交互协议说明

#### Header头参数
| Paremter      | Requerd | Type  | 说明               |
| :----         | :---    |:----- | -----             |
| channel       | Y       |long   | 分配的渠道ID        |
| timestamp     | Y       |long   | 当前毫秒时间戳       |
| nonce         | Y       |string | 八位长度随机字符串    |
| sign          | Y       |string | 签名                |

#### 签名算法如下：
> 下述案例以渠道ID=C100000, 渠道密钥=7a01637f972c4815a8406101463f91c1为例, 接入时以实际分配的数据为准.

#####  第一步
分别拼channel,nonce,timestamp; 键值之间用"="链接, 键之间用"&"连接, 得到字符串

```text
channel=C100000&nonce=e2dd305d79c2ebc8&timestamp=1605165911312
```

##### 第二步
拼接分配的渠道密钥, 得到

```text
channel=C100000&nonce=e2dd305d79c2ebc8&timestamp=16051659113127a01637f972c4815a8406101463f91c1
```

##### 第三步
计算上述字符串的32位md5, 作为sign值, 放入header头

```text
MD5(channel=C100001&nonce=z2q4bmke&timestamp=16087271730007a01637f972c4815a8406101463f91c1)
===> 2ebe68266d0d32fd8ee9027dd205a1af
```
#### 各个环境数据接口地址

| 环境           | 接口地址                              |
| ------------  | -----------------------------        |
| 正式环境       | http://api-delivery.xxxxxxx.com      |

## 二. 接口定义
### 1. 邮政订单提交

#### a. 请求方式

POST  /api/order/submit

#### b. 参数说明

###### 请求参数

| Paramter          | Type    | Re   | Description       |
| -------------     | ------  | -----| ----------------- |
| product_code      | string  |  *   | 产品代码           |
| insurance_flag    | string  |  -   | 保险保价标志        |
| insurance_amount  | decimal |  -   | 保价保险金额  单位：元|
| clcttype          | integer |  -   | 揽收类型            |
| untread           | string  |  -   | 退回类型            |
| total_weight      | Integer |  *   | 运单总重量（克）      |
| vol_weight        | Integer |  -   | 体积重量，长×宽×高/6000,单位：厘米  |
| length            | Integer |  -   | 长                 |
| width             | Integer |  -   | 宽                 |
| height            | Integer |  -   | 高                  |
| remark            | string  |  -   | 备注                |
| good_nature       | string  |  -   | 内件性质             |
| declaration_type  | string  |  -   | 报关种类             |
| sender            | Sender  |  *   | 发件人信息             |
| receiver          | Receiver|  *   | 收件人信息             |
| items             | array&lt;Item>  |  *   | 商品列表             |


| 发件人信息(Sender) |    |     |       |
| -------- | -------- | -------- | ------------ |
| name     | string   | *      | 姓名（英文） |
| postcode | string   | *       | 邮编         |
| phone    | string   | *       | 电话         |
| country  | string   | *       | 国家（CN）   |
| province | string   | *       | 省份（英文） |
| city     | string   | *       | 城市（英文） |
| county   | string   | *       | 区县（英文） |
| company  | string   | -       | 公司（英文） |
| street   | string   | *       | 街道（英文） |
| email    | string   | -       | 电子邮箱     |

| 收件人信息(Receiver) |    |     |       |
| -------- | -------- | -------- | --------------- |
| name     | string   | *       | 姓名            |
| postcode | string   | *       | 邮编            |
| phone    | string   | -       | 电话            |
| country  | string   | *       | 国家 (国家简码) |
| province | string   | *       | 省份（洲）      |
| city     | string   | *       | 城市            |
| company  | string   | -       | 公司            |
| street   | string   | *       | 街道            |
| houseno  | string   | -       | 门牌号          |
| email    | string   | -       | 电子邮箱        |

| 商品项(Item)  |         |      |                          |
| ------------  | ------- | ---- | ------------------------ |
| cnname        | string  | *    | 中文名称                 |
| enname        | string  | *    | 英文名称                 |
| count         | integer | *    | 数量                     |
| unit          | string  | -    | 计量单位（默认为‘个’）   |
| weight        | integer | *    | 单个商品重量，单位克     |
| delcare_value | money   | *    | 单个商品申报价格（美元） |
| origin        | string  | -    | 原产地（默认为CN）       |
| description   | string  | -    | 描述（英文）             |
| taxation_id   | string  | -    | 税则号                   |
| sell_url      | string  | -    | 销售链接                 |

###### 返回数据

| Response         |         |                     |
| --------------   | ------- | ------------------- |
| code             | int     |                     |
| data             | object  | 返回数据             |
| data.orderno     | string  | 订单号               |
| data.mailno      | string  | 邮政号               |


#### c. 请求示例
```_
POST ${Host}/api/order/submit
content-type: application/json
channel: C100000
nonce: e2dd305d79c2ebc8
timestamp: 1605165911312
sign: 2ebe68266d0d32fd8ee9027dd205a1af

{}
```

#### d. 返回示例
```JSON
{
	"code": 0,
	"data": {
		"orderno": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
		"mailno": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
	}
}
```

### 2. 邮政邮件轨迹提交

#### a. 请求方式

POST  /api/trace/submit

#### b. 参数说明

###### 请求参数

| Parameter        | Type   | Re  | Desc                              |
| ---------------- | ------ | --- | -------------------------------   |
| type             | String | *   | 消息类型：  I,U,D                  |
| mailno           | String | *   | 邮件号                             |
| event_id         | String | -   | 事件ID, 为空时以系统填充ID作为事件ID |
| event_time       | String | *   | 事件发生时间:  yyyy-MM-dd HH:mm:ss  |
| event_time_zone  | String | *   | 时区：例如+8                       |
| event_status     | String | *   | 事件状态                           |
| event_country    | String | -   | 发生国家：国家名称，直接展示          |
| event_city       | String | -   | 发生城市：New York，直接展示         |
| deal_org_name    | String | -   | 处理机构名称                        |
| deal_org_en_name | String | -   | 处理机构英文名称                    |
| deal_org_phone   | String | -   | 处理机构联系电话                    |
| dest_city        | String | -   | 目的地城市（离开某地发往dest_city）   |
| sender_city      | String | -   | 寄件城市                           |
| receiver_city    | String | -   | 收件城市                           |
| signer_name      | String | -   | 签收人名称                         |
| undelivery_cause | String | -   | 投递失败原因                       |

###### 注意事项

邮政方数据排重条件mailno、event_id、event_time
如需修改时间，请删除原有的消息，重新传一个新的
如果修改其他内容，直接修改

###### 返回数据

| Response         |         |                     |
| --------------   | ------- | ------------------- |
| code             | int     |                     |
| success          | boolean |                     |
| data             | object  | 返回数据             |
| data.trace_id    | string  | 轨迹数据ID, 用户查询   |

#### c. 请求示例
```_
POST ${Host}/api/trace/submit
content-type: application/json
channel: C100001
nonce: e2dd305d79c2ebc8
timestamp: 1582268384295
sign: oOS98eZJI2KDzoOlfl7Je0XN4gAyrqNG

{
  "type": "D",
  "mailno": "A1111111",
  "event_time": "2020-11-06 22:13:00",
  "event_time_zone": "+8",
  "event_status": "O_111",
  "event_country": "CN"
}
```

#### d. 返回示例
```JSON
{
	"code": 0,
	"success": true,
	"data": {
		"trace_id": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
	}
}
```

### 3. 邮政邮件轨迹查询

#### a. 请求方式

GET  ${Host}/api/trace/{trace_id}

#### b. 参数说明

###### 返回数据

| Response              |         |                     |
| --------------        | ------- | ------------------- |
| code                  | int     |                     |
| success               | boolean |                     |
| data                  | object  | 返回数据             |
| data.trace_id         | long    | 轨迹数据ID           |
| data.type             | string  | 消息类型             |
| data.mailno           | string  | 邮件号               |
| data.event_id         | string  | 事件ID               |
| data.orderno          | long    | 订单号               |
| data.event_time       | string  | 事件时间              |
| data.event_time_zone  | string  | 事件时区              |
| data.event_status     | string  | 事件状态              |
| data.event_country    | string  | 事件国家              |
| data.event_city       | string  | 事件城市              |
| data.deal_org_name    | string  | 处理机构名称           |
| data.deal_org_en_name | string  | 处理机构英文名称        |
| data.deal_org_phone   | string  | 处理机构联系方式        |
| data.dest_city        | string  | 目的地城市             |
| data.sender_city      | string  | 寄件城市               |
| data.receiver_city    | string  | 收件城市               |
| data.signer_name      | string  | 签收人名称              |
| data.undelivery_cause | string  | 投递失败原因            |
| data.create_time      | long    | 数据创建时间            |
| data.status           | int     | 数据状态               |

#### c. 请求示例
```_
GET ${Host}/api/trace/1111111111
content-type: application/json
channel: C100001
nonce: e2dd305d79c2ebc8
timestamp: 1582268384295
sign: oOS98eZJI2KDzoOlfl7Je0XN4gAyrqNG
```

#### d. 返回示例
```JSON
{
  "code": 0,
  "success": true,
  "data": {
    "id": 774110505139699712,
    "channel": 100001,
    "batchno": 101892,
    "type": "I",
    "mailno": "A1111111",
    "orderno": null,
    "event_time": "2020-11-06 22:13:00 333",
    "event_time_zone": "+8",
    "event_status": "O_111",
    "event_country": "CN",
    "event_city": null,
    "deal_org_name": null,
    "deal_org_en_name": null,
    "deal_org_phone": null,
    "dest_city": null,
    "sender_city": null,
    "receiver_city": null,
    "signer_name": null,
    "undelivery_cause": null,
    "create_time": 1604632726703,
    "status": 1
  }
}
```

### 4. 邮政号下轨迹数据获取

#### a. 请求方式

GET  ${Host}/api/trace/query_by_mailno/{mailno}

#### b. 参数说明

###### 返回数据

| Response              |         |                     |
| --------------        | ------- | ------------------- |
| code                  | int     |                     |
| success               | boolean |                     |
| data                  | array   | 返回数据             |
| data.trace_id         | long    | 轨迹数据ID           |
| data.type             | string  | 消息类型             |
| data.mailno           | string  | 邮件号               |
| data.event_id         | string  | 事件ID               |
| data.orderno          | long    | 订单号               |
| data.event_time       | string  | 事件时间              |
| data.event_time_zone  | string  | 事件时区              |
| data.event_status     | string  | 事件状态              |
| data.event_country    | string  | 事件国家              |
| data.event_city       | string  | 事件城市              |
| data.deal_org_name    | string  | 处理机构名称           |
| data.deal_org_en_name | string  | 处理机构英文名称        |
| data.deal_org_phone   | string  | 处理机构联系方式        |
| data.dest_city        | string  | 目的地城市             |
| data.sender_city      | string  | 寄件城市               |
| data.receiver_city    | string  | 收件城市               |
| data.signer_name      | string  | 签收人名称              |
| data.undelivery_cause | string  | 投递失败原因            |
| data.create_time      | long    | 数据创建时间            |
| data.status           | int     | 数据状态               |

#### c. 请求示例
```_
GET ${Host}/api/trace/query_by_mailno/AQ850000084CN
content-type: application/json
channel: C100001
nonce: e2dd305d79c2ebc8
timestamp: 1582268384295
sign: oOS98eZJI2KDzoOlfl7Je0XN4gAyrqNG
```

#### d. 返回示例
```JSON
{
  "code": 0,
  "success": true,
  "data": [{
    "id": 774110505139699712,
    "channel": 100001,
    "batchno": 101892,
    "type": "I",
    "mailno": "A1111111",
    "orderno": null,
    "event_time": "2020-11-06 22:13:00 333",
    "event_time_zone": "+8",
    "event_status": "O_111",
    "event_country": "CN",
    "event_city": null,
    "deal_org_name": null,
    "deal_org_en_name": null,
    "deal_org_phone": null,
    "dest_city": null,
    "sender_city": null,
    "receiver_city": null,
    "signer_name": null,
    "undelivery_cause": null,
    "create_time": 1604632726703,
    "status": 1
  }]
}
```

### 5. 邮政邮件面单获取

#### a. 请求方式

GET  ${Host}/api/order/pdf/{mailno}

#### b. 参数说明

###### 返回数据

| Response              |         |                     |
| --------------        | ------- | ------------------- |
| code                  | int     |                     |
| success               | boolean |                     |
| data                  | object  | 返回数据             |
| data.content          | long    | PDF文件Base64编码    |

#### c. 请求示例
```_
GET http://localhost:9090/api/order/pdf/CE000377256CN
content-type: application/json
channel: C100000
nonce: e2dd305d79c2ebc8
timestamp: 1605165911312
sign: 2ebe68266d0d32fd8ee9027dd205a1af
```

#### d. 返回示例
> 示例中文本长度已缩减

```JSON
{
  "code": 0,
  "success": true,
  "data": {
    "content": "JVBERi0xLjQKJeLjz9MKNiAwIG9iago8PC9GaWx0ZXIvRmxhdGVEZWNvZGUvTGVuZ3RoIDU1Pj5zdHJgMTk+PgolaVRleHQtNS4zLjIKc3RhcnR4cmVmCjMyMzc2CiUlRU9GCg=="
  }
}
```

### e.PDF保存本地
```JAVA
String targetPath = "/data/pdf/xxxx.pdf";
// 调用接口获取结果数据
InnetResult<MailReceipt> innetResult = InnetBusinessManager.getInstance().getReceipt(mailnum);
// 返回成功则写入文件
if (innetResult.isSuccess()) {
	String base64code = innetResult.getData().getContent();
	Base64.Decoder decoder = Base64.getDecoder();
	byte[] buffer = decoder.decode(base64code);
	FileOutputStream out = new FileOutputStream(targetPath);
	out.write(buffer);
	out.close();
	return true;
}
return false
```

## 三. 枚举类型

#### 产品代码
- 01：E速宝优先普货
- 02：E速宝优先特货
- 03：E速宝经济普货
- 04：E速宝经济特货
- 05：E速宝特惠普货
- 06：E速宝特惠特货

#### 保险保价标志
- 1:基本
- 2:保价
- 3:报险

#### 揽收类型，
- 0：用户自送(默认)
- 1：上门揽收

#### 退回类型
- 01：退回(默认)
- 02：丢弃
- 03：保证支付退回的邮件

#### 内件性质
- 00：一般商品
- 01：带电
- 02：其他敏感

#### 报关种类
- 00：一般报关
- 01：9610报关

#### 国家短码
可参考  [iso-3166-1](http://doc.chacuo.net/iso-3166-1)

#### 事件状态
- O_007: 转运中
- O_008: 到达
- O_009: 离开
- O_010: 已投递
- O_011: 退回
- O_012: 拒收
- O_013: 投递失败
- O_014: 出门投递
- O_016: 已妥投
- O_017: 到达待取
- O_018: 妥投

#### 投递失败原因
- 1: 正在投递中
- 10: 收件人名址有误
- 11: 查无此人
- 12: 收件人不在指定地址
- 13: 拒收退回
- 14: 收件人要求延迟投递
- 16: 误投
- 17: 邮件错发
- 18: 收到时破损，无法投出
- 19: 禁寄物品
- 20: 限寄物品
- 21: 待收费后
- 22: 无人认领
- 23: 无法找到收件人
- 24: 因不可抗力原因，邮件未投出
- 25: 收件人要求自取
- 26: 法定假日，无法投递
- 27: 邮件丢失
- 28: 人已他往
- 29: 收件人有信箱
- 30: 安排投递
- 31: 正在投递
- 32: 查无此单位
- 33: 地址不祥，无电话，电话不对
- 34: 地址不详
- 35: 撤回
- 36: 迁移新址不明
- 37: 逾期未领
- 40: 等待退回仓库或发货方
- 81: 投递到包裹站
- 82: 逾期投递员收回
- 99: 其它



## 四. 错误码索引

##### 通用 #####
|code         |HTTP STATUS    | DESC            | TODO           |
|:----        |:-------       |:---             | --             |
|10000        |200            | 服务端内部异常    | 联系管理员       |
|10001        |200            | 签名校验失败      | -              |
|10002        |200            | 无效参数         | -              |
|20000        |200            | 邮政接口返回错误  | -              |
