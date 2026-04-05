# 百居易 API 开放接口文档

> 来源: 飞书 Wiki - https://my.feishu.cn/wiki/NFdawEiAEiHXZ2kMaBVcKHxBnze

---

# 协同流程

![配图](image:NgOBbSYGgo2yBrxf3jQcBRKUnHf)

---

# 接口认证

## 认证要素

AppID 为系统分配给合作伙伴的唯一应用ID。

SecretKey 为系统分配的秘钥。

## 举例说明

AppID: myapp

SecretKey: mysecre

生成认证字串: "Basic " + Base64Encode("myapp:mysecret")

结果置于请求header中: Authorization: Basic bXlhcHA6bXlzZWNyZXQ=

---

# 关于令牌

令牌分为AccessToken和RefreshToken。与`接口认证`不同的是，令牌用于获取房东账号的业务数据。

## AccessToken

访问令牌。用于获取授权房东账号的业务数据，令牌有效时间24小时。

## RefreshToken

刷新令牌。用于获取新的访问令牌，刷新后原令牌失效。

## 访问权限认证

百居易API的访问权限认证包括Hostex-Operator-Id和Hostex-Access-Token两个必须的http头。

> OperatorId是百居易房东账号的唯一id，获取方式请参考`令牌与授权`部分。

---

# 令牌与授权

## 查询刷新令牌 /refresh_token/query

**请求方式**

POST

**请求示例**

curl -X POST https://api.myhostex.com/refresh_token/query \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Basic bXlhcHA6bXlzZWNyZXQ='

**响应示例**

```json
{
    "request_id": "UNKNOWN_ID",
    "error_code": 0,
    "error_msg": "success",
    "data": {
        "refresh_token": "W7NmYI1XcqLrcK4nqiGjIrfhfUofZCET1N5A1jh2",
        "refresh_token_expire": "2024-05-05 12:15:00"
    }
}
```

**响应说明**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_fiqrAv）

---

## 获取刷新令牌 /refresh_token/refresh

**请求方式**

POST

> 每次获取旧的令牌会失效

**请求示例**

curl -X POST https://api.myhostex.com/refresh_token/refresh \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Basic bXlhcHA6bXlzZWNyZXQ='

**响应示例**

```json
{
    "request_id": "UNKNOWN_ID",
    "error_code": 0,
    "error_msg": "success",
    "data": {
        "refresh_token": "W7NmYI1XcqLrcK4nqiGjIrfhfUofZCET1N5A1jh2",
        "refresh_token_expire": "2024-05-05 12:15:00"
    }
}
```

**响应说明**

参考：查询刷新令牌 /refresh_token/query

---

## 获取访问令牌 /access_token/refresh

**请求方式**

POST

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_um8LSK）

> 每次获取旧的access token会失效

**请求示例**

curl -X POST https://api.myhostex.com/access_token/refresh \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Basic MTbXlhcHA6bXlzZWNyZXQ=' \
  -d '{"refresh_token":"W7NmYI1XcqLrcK4nqiGjIrfhfUofZCET1N5A1jh2"}'

**响应示例**

```json
{
    "request_id": "UNKNOWN_ID",
    "error_code": 0,
    "error_msg": "success",
    "data": {
        "access_token": "89ed05bd6a2907706e80fdaa83c32d302325f30b",
        "expire_time": "2019-05-05 14:28:03"
    }
}
```

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_OAnPfs）

---

## 新增房东授权 /manage_operator/join

**请求方式**

POST

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_mejo7p）

**请求示例**

curl -X POST https://api.myhostex.com/manage_operator/join \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Basic bXlhcHA6bXlzZWNyZXQ=' \
  -d '{"account":"188117728812","password":"wwp12345"}'

**响应示例**

```json
{
    "request_id": "UNKNOWN_ID",
    "error_code": 0,
    "error_msg": "",
    "data": {
        "operator_id": 10035,
        "account": "1290059209@qq.com"
    }
}
```

**响应说明**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_l2sGFy）

---

# 房间和房型

# 房间

## 房间列表 /house/list

**请求方式**

GET

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_DhrpcD）

**请求示例**

curl https://api.myhostex.com/house/list?page=1&page_size=1 \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Operator-Id: 10008' \
  -H 'Hostex-Access-Token: SmpZgfDwMf91S6V7mYw9gqColimpFb7W56pPCYy9F4'

**响应示例**

```json
{
    "request_id": "UNKNOWN_ID",
    "error_code": 0,
    "error_msg": "",
    "data": {
        "list": [
            {
                "id": 36,
                "type": 6,
                "space_type": 2,
                "area": 50,
                "private_bathroom_count": 1,
                "public_bathroom_count": 1,
                "living_room_count": 1,
                "study_count": 0,
                "balcony_count": 1,
                "kitchen_count": 1,
                "bed_sheet_change": 1,
                "title_alias": "123",
                "street": "北园一区",
                "building_name": "金台园",
                "house_number": "2 号楼 2 单元 601",
                "longitude": 116.632613,
                "latitude": 40.324267,
                "house_descriptions": [
                    {
                        "id": 38,
                        "house_id": 36,
                        "locale": "zh",
                        "title": "森林小屋 123",
                        "description": "四季宜人"
                    }
                ],
                "house_prices": [
                    {
                        "id": 4110,
                        "house_id": 36,
                        "thirdparty_account_id": 0,
                        "thirdparty_type": 6,
                        "origin_house_id": "36",
                        "currency_type": "CNY",
                        "daily_price": 0,
                        "cleaning_fee": 0,
                        "cash_pledge": 0,
                        "extra_tenant_price": 0,
                        "weekend_price": 0,
                        "extra_tenant_min_count": 0,
                        "sync_status": 0,
                        "is_shelf": 1
                    }
                ],
                "house_cover_picture": {
                    "id": 9019,
                    "house_id": 36,
                    "details": "{\"filename\":\"YEK1540903574642.png\",\"original_url\":\"https:\\/\\/oss.image.xiaogetech.com\\/YEK1540903574642.png\",\"small_url\":\"https:\\/\\/oss.image.xiaogetech.com\\/YEK1540903574642.png\\/small\",\"medium_url\":\"https:\\/\\/oss.image.xiaogetech.com\\/YEK1540903574642.png\\/medium\",\"large_url\":\"https:\\/\\/oss.image.xiaogetech.com\\/YEK1540903574642.png\\/large\",\"extra_large_url\":\"https:\\/\\/oss.image.xiaogetech.com\\/YEK1540903574642.png\\/xlarge\",\"extra_extra_large_url\":\"https:\\/\\/oss.image.xiaogetech.com\\/YEK1540903574642.png\\/xxlarge\"}",
                    "caption": "",
                    "order": 0
                }
            }
        ],
        "total": 113
    }
}
```

返回字段说明

请参考房间详情接口

---

## 房间详情 /house/detail

**请求方式**

GET

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_lHELxX）

**请求示例**

curl https://api.myhostex.com/house/detail?house_id=3115 \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Operator-Id: 10008' \
  -H 'Hostex-Access-Token: SmpZgfDwMf91S6V7mYw9gqColimpFb7W56pPCYy9F4'

**响应示例**

```json
{
    "request_id": "UNKNOWN_ID",
    "error_code": 0,
    "error_msg": "success",
    "data": {
        "house": {
            "id": 36,
            "type": 6,
            "space_type": 2,
            "area": 50,
            "private_bathroom_count": 1,
            "public_bathroom_count": 1,
            "living_room_count": 1,
            "study_count": 0,
            "balcony_count": 1,
            "kitchen_count": 1,
            "bed_sheet_change": 1,
            "title_alias": "123",
            "street": "北园一区",
            "building_name": "金台园",
            "house_number": "2 号楼 2 单元 601",
            "longitude": 116.632613,
            "latitude": 40.324267,
            "house_descriptions": [
                {
                    "id": 38,
                    "locale": "zh",
                    "title": "森林小屋 123",
                    "description": "四季宜"
                }
            ],
            "house_prices": [
                {
                    "thirdparty_type": 0,
                    "origin_house_id": "36",
                    "currency_type": "CNY",
                    "daily_price": 500,
                    "cleaning_fee": 0,
                    "cash_pledge": 0,
                    "extra_tenant_price": 0,
                    "weekend_price": 0,
                    "is_shelf": 1
                }
            ],
            "house_pictures": [
                {
                    "id": 9019,
                    "house_id": 36,
                    "details": "{\"filename\":\"YEK1540903574642.png\",\"original_url\":\"https:\\/\\/oss.image.xiaogetech.com\\/YEK1540903574642.png\"}",
                    "caption": "",
                    "order": 0
                }
            ]
        }
    }
}
```

**返回字段说明**

**house:** （详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_FUea1J）

**house_descriptions[]:** （详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_h2rQDo）

**prices[]:** （详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_jQ128I）

**house_pictures[]:** （详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_mkR3jC）

---

# 房型

## 检索房型 house_type/list

说明：结果不返回没有关联渠道房源的房型

**请求方式**

GET

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_cf7vnb）

**请求示例**

curl -X GET \
  'https://api.myhostex.com/house_type/list?page=1&page_size=10' \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Access-Token: 817266660a95c28c6f6b1764a186d03de1091b3a' \
  -H 'Hostex-Operator-Id: 10008' \
  -H 'cache-control: no-cache'

**响应示例**

```json
{
    "request_id": "RT1587980707619918",
    "error_code": 0,
    "error_msg": "success",
    "data": {
        "list": [
            {
                "id": 94,
                "title": "001-大床单间【房型】",
                "create_time": "2019-10-10 18:39:18",
                "house_type_prices": [
                    {
                        "thirdparty_type": 7,
                        "origin_house_id": "8083661",
                        "daily_price": 3000,
                        "weekend_price": 3800,
                        "is_shelf": 1
                    }
                ],
                "houses": [
                    {
                        "house_id": 11085220,
                        "house_title": "阳光沙滩的金色光芒，放松心灵的休憩之地"
                    },
                    {
                        "house_id": 11085322,
                        "house_title": "北京中式风格房间"
                    }
                ]
            }
        ],
        "total": 1
    }
}
```

**返回字段说明**

list[] （详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_HhsE3W）

house_type_prices[] 房型关联的渠道房源列表 （详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_dKmLOw）

houses[] 房型关联的房间列表 （详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_bkL2r1）

---

# 房态和房价

## 检索日历 /calendar/query

**请求方式**

GET

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_jxbzdp）

**请求示例**

curl -X GET \
  'https://api.myhostex.com/calendar/query?house_type_ids=[94]&begin=2020-5-10&end=2020-5-10' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -H 'Hostex-Access-Token: 817266660a95c28c6f6b1764a186d03de1091b3a' \
  -H 'Hostex-Operator-Id: 10008'

**响应示例**

```json
{
    "request_id": "RT1587982874779758",
    "error_code": 0,
    "error_msg": "success",
    "data": {
        "calendar_list": [
            {
                "house_type_id": 94,
                "calendar": [
                    {
                        "house_type_id": 94,
                        "date": "2020-05-10",
                        "day_of_week": 0,
                        "type": "default",
                        "prices": [
                            {
                                "thirdparty_type": 7,
                                "origin_house_id": "8083661",
                                "price": 3000,
                                "available": 5
                            }
                        ]
                    }
                ]
            }
        ]
    }
}
```

**返回字段说明**

calendar_list[]: （详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_W7UFFU）

calendar[]: （详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_trwM1d）

prices[]: （详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_YPuX9y）

---

## 设置房态 calendar/set_available

**请求方式**

POST

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_dTvQrj）

### 一、当修改房间房态时

#### 1.1 若不传thirdparty_type

会默认修改百居易本地房态（房态页面所展示的房态），并且会改变房间关联所有渠道房源的房态。若房间归属某一房型，会触发对应房型的库存联动操作，并且会改变房型关联所有渠道房源的库存。

**请求示例**

curl -X POST https://api.myhostex.com/calendar/set_available \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Access-Token: 817266660a95c28c6f6b1764a186d03de1091b3a' \
  -H 'Hostex-Operator-Id: 10008' \
  -H 'cache-control: no-cache' \
  -d '{"house_id":3416,"begin":"2020-5-10","end":"2020-5-10","available":0}'

**响应示例**

```json
{
    "request_id": "RT1587978162870881",
    "error_code": 0,
    "error_msg": "",
    "data": []
}
```

#### 1.2 若传thirdparty_type

则修改指定渠道房态（不会修改百居易本地房态），此时会针对指定的房间的指定渠道的所有房源进行修改。

**请求示例**

curl -X POST https://api.myhostex.com/calendar/set_available \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Access-Token: 817266660a95c28c6f6b1764a186d03de1091b3a' \
  -H 'Hostex-Operator-Id: 10008' \
  -H 'cache-control: no-cache' \
  -d '{"house_id":3416,"begin":"2020-5-10","end":"2020-5-10","available":0,"thirdparty_type":0}'

**响应示例**

```json
{
    "request_id": "RT1587978554547466",
    "error_code": 0,
    "error_msg": "操作成功",
    "data": {
        "task": [
            {
                "task_id": 55294,
                "thirdparty_type": "0",
                "origin_house_id": "36241301"
            },
            {
                "task_id": 55295,
                "thirdparty_type": "0",
                "origin_house_id": "31094407"
            }
        ]
    }
}
```

#### 1.3 指定渠道并指定渠道房源id

（注意：渠道类型参数thirdparty_type不能遗漏）

**请求示例**

curl -X POST https://api.myhostex.com/calendar/set_available \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Access-Token: 817266660a95c28c6f6b1764a186d03de1091b3a' \
  -H 'Hostex-Operator-Id: 10008' \
  -H 'cache-control: no-cache' \
  -d '{"house_id":3416,"begin":"2020-5-10","end":"2020-5-10","available":0,"thirdparty_type":0,"origin_house_id":"36241301"}'

### 二、修改房型关联渠道时

与房间类似，支持改所有、改指定渠道、以及指定渠道房源id。修改房型的库存不会改变百居易房间的本地房态，只会修改渠道房源的库存。

**请求示例**

curl -X POST https://api.myhostex.com/calendar/set_available \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Access-Token: 817266660a95c28c6f6b1764a186d03de1091b3a' \
  -H 'Hostex-Operator-Id: 10008' \
  -H 'cache-control: no-cache' \
  -d '{"house_type_id":94,"begin":"2020-5-10","end":"2020-5-10","available":3}'

**响应示例**

```json
{
    "request_id": "RT1587979084019102",
    "error_code": 0,
    "error_msg": "操作成功",
    "data": {
        "task": [
            {
                "task_id": 55297,
                "thirdparty_type": "7",
                "origin_house_id": "8083661"
            }
        ]
    }
}
```

---

## 设置价格 /calendar/set_price

**请求方式**

POST

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_6WzFNL）

**请求示例**

curl -X POST https://api.myhostex.com/calendar/set_price \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Access-Token: 817266660a95c28c6f6b1764a186d03de1091b3a' \
  -H 'Hostex-Operator-Id: 10008' \
  -H 'cache-control: no-cache' \
  -d '{"house_id":3416,"begin":"2020-5-1","end":"2020-5-4","daily_price":2000}'

**响应示例**

```json
{
    "request_id": "RT1587979530987690",
    "error_code": 0,
    "error_msg": "操作成功",
    "data": {
        "task": [
            {
                "task_id": 55298,
                "thirdparty_type": "10",
                "origin_house_id": "3416"
            },
            {
                "task_id": 55299,
                "thirdparty_type": "6",
                "origin_house_id": "3416"
            },
            {
                "task_id": 55300,
                "thirdparty_type": "0",
                "origin_house_id": "36241301"
            },
            {
                "task_id": 55301,
                "thirdparty_type": "0",
                "origin_house_id": "31094407"
            }
        ]
    }
}
```

在改价时可以通过参数控制修改范围：

1. `house_type_id` or `house_id`
2. `house_type_id` or `house_id` + `thirdparty_type`
3. `house_type_id` or `house_id` + `thirdparty_type` + `origin_house_id`

按此顺序，修改范围逐级缩小。

---

# 订单

## 检索订单 /reservation/query

**请求方式**

GET

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_9WHqNR）

**请求示例**

curl https://api.myhostex.com/reservation/query?page=1&page_size=1 \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Operator-Id: 10008' \
  -H 'Hostex-Access-Token: SmpZgfDwMf91S6V7mYw9gqColimpFb7W56pPCYy9F4'

**响应示例**

```json
{
    "request_id": "UNKNOWN_ID",
    "error_code": 0,
    "error_msg": "success",
    "data": {
        "list": [
            {
                "code": "1-801252832144",
                "uniq_code": "1-801252832144-AJS82",
                "reservation_id": "1-801252832144",
                "house_id": 2866,
                "thirdparty_type": 1,
                "origin_house_id": "1956703",
                "number_of_guests": 1,
                "check_in": "2018-10-10",
                "check_out": "2018-10-11",
                "check_in_time": "00:00:00",
                "check_out_time": "00:00:00",
                "booked_time": "2018-07-13 14:53:47",
                "cancel_rule": 0,
                "status": "accepted",
                "staying_status": "wait_stay",
                "create_time": "2018-07-13 14:56:07",
                "update_time": "2019-04-16 10:28:35",
                "thread_id": "1-p2p-8795560-9043646",
                "in_box": 0,
                "lock_password": "123456",
                "price_items": [
                    {"title": "房费", "type": "ACCOMMODATION", "currency_type": "CNY", "price": 60},
                    {"title": "佣金", "type": "HOST_SERVICE_FEE", "currency_type": "CNY", "price": -10},
                    {"title": "预计收入", "type": "HOST_EARNINGS", "currency_type": "CNY", "price": 60},
                    {"title": "平台补贴", "type": "PLATFORM_SUBSIDIES", "currency_type": "CNY", "price": 10}
                ],
                "guests": [
                    {
                        "name": "小黑",
                        "full_name": "小黑 韩",
                        "email": "user-sv8t6u97bzi04aq5@guest.airbnb.com",
                        "phone": "+86 186 2961 6237",
                        "photo": "https://z1.muscache.cn/im/pictures/user/e7de023c-2cf3-4ce6-b23d-6bf580d23bf7.jpg?aki_policy=profile_x_medium"
                    },
                    {
                        "name": "丽",
                        "full_name": "丽 谢",
                        "email": "user-5ncghzp65zbygafi@guest.airbnb.com",
                        "phone": "+86 134 7701 9836",
                        "photo": "https://z1.muscache.cn/im/pictures/user/1a2b5b38-fa06-4a7c-a5ee-99ebbcef0c31.jpg?aki_policy=profile_x_medium"
                    }
                ]
            }
        ]
    }
}
```

**字段说明**

参考订单详情

---

## 订单详情 /reservation/detail

**请求方式**

GET

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_zcFPqh）

**请求示例**

curl https://api.myhostex.com/reservation/detail?reservation_code=1-801252832144 \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Operator-Id: 10008' \
  -H 'Hostex-Access-Token: SmpZgfDwMf91S6V7mYw9gqColimpFb7W56pPCYy9F4'

**响应示例**

```json
{
    "request_id": "UNKNOWN_ID",
    "error_code": 0,
    "error_msg": "success",
    "data": {
        "reservation": {
            "code": "1-801252832144",
            "uniq_code": "1-801252832144-AJS82",
            "reservation_id": "1-801252832144",
            "house_id": 2866,
            "thirdparty_type": 1,
            "origin_house_id": "1956703",
            "number_of_guests": 1,
            "check_in": "2018-10-10",
            "check_out": "2018-10-11",
            "check_in_time": "00:00:00",
            "check_out_time": "00:00:00",
            "booked_time": "2018-07-13 14:53:47",
            "cancel_rule": 0,
            "status": "accepted",
            "staying_status": "wait_stay",
            "create_time": "2018-07-13 14:56:07",
            "update_time": "2019-04-16 10:28:35",
            "thread_id": "1-p2p-8795560-9043646",
            "in_box": 0,
            "lock_password": "123456",
            "price_items": [
                {"title": "房费", "type": "ACCOMMODATION", "currency_type": "CNY", "price": 60},
                {"title": "佣金", "type": "HOST_SERVICE_FEE", "currency_type": "CNY", "price": -10},
                {"title": "预计收入", "type": "HOST_EARNINGS", "currency_type": "CNY", "price": 60},
                {"title": "平台补贴", "type": "PLATFORM_SUBSIDIES", "currency_type": "CNY", "price": 10}
            ],
            "guests": [
                {
                    "name": "小黑",
                    "full_name": "小黑 韩",
                    "email": "user-sv8t6u97bzi04aq5@guest.airbnb.com",
                    "phone": "+86 186 2961 6237",
                    "photo": "https://z1.muscache.cn/im/pictures/user/e7de023c-2cf3-4ce6-b23d-6bf580d23bf7.jpg?aki_policy=profile_x_medium"
                }
            ]
        }
    }
}
```

**字段说明**

**reservation:** （详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_rEFwVr）

**price_items[]:** （详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_qVMv17）

**guests[]:**

> 若存在多个房客信息，则第一位房客为预订人，其余为入住人。
> 关于房客姓名，可优先使用`full_name`，若`full_name`为空则使用`name`。目前爱彼迎和小猪短租有全名。

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_AiUjLg）

---

## 接受订单 /reservation/approve

**请求方式**

POST

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_55rhAw）

**请求示例**

curl -X POST https://api.myhostex.com/reservation/approve \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Operator-Id: 100008' \
  -H 'Hostex-Access-Token: SmpZgfDwMf91S6V7mYw9gqColimpFb7W56pPCYy9F4' \
  -d '{"reservation_code":"1-801252832144"}'

**响应参数**

```json
{
   "error_code": 0,
   "error_msg": "success",
   "request_id": "RT15566009870276510"
}
```

```json
{
   "error_code": 200000,
   "error_msg": "此订单已不是待确认状态",
   "request_id": "RT15566009870276510"
}
```

---

## 拒绝订单 /reservation/decline

**请求方式**

POST

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_NtTO5i）

**请求示例**

curl -X POST https://api.myhostex.com/reservation/decline \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Operator-Id: 100008' \
  -H 'Hostex-Access-Token: SmpZgfDwMf91S6V7mYw9gqColimpFb7W56pPCYy9F4' \
  -d '{"reservation_code":"1-801252832144"}'

**响应参数**

```json
{
   "error_code": 0,
   "error_msg": "success",
   "request_id": "RT15566009870276510"
}
```

---

## 创建手工单 /reservation/create

> ⚠️ 请谨慎操作，手工单只能创建和取消，无法删除。

**请求方式**

POST

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_Frsd4n）

**请求示例**

curl -X POST 'https://api.myhostex.com/reservation/create' \
  --header 'Hostex-Operator-Id: 10008' \
  --header 'Hostex-Access-Token: SmpZgfDwMf91S6V7mYw9gqColimpFb7W56pPCYy9F4' \
  --header 'Content-Type: application/json' \
  --data-raw '{
    "house_id": "123456",
    "channel_name": "自来客",
    "check_in_date": "2025-01-01",
    "check_out_date": "2025-01-02",
    "number_of_guests": "1",
    "guest_name": "张三",
    "mobile": "18888888888",
    "email": "at@xiaogetest.com",
    "currency": "CNY",
    "rate_amount": "10",
    "commission_amount": "1",
    "received_amount": "0",
    "income_method_name": "其他",
    "remarks": "123456"
  }'

**响应参数**

```json
{
    "request_id": "RT202507111015382300",
    "error_code": 0,
    "error_msg": "",
    "data": {
        "reservation_code": "5-67LK8YBI2"
    }
}
```

---

## 取消手工单 /reservation/cancel

> ⚠️ 只能取消手工单，渠道订单无法取消。

**请求方式**

POST

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_kAPjof）

**请求示例**

curl -X POST https://api.myhostex.com/reservation/cancel \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Operator-Id: 100008' \
  -H 'Hostex-Access-Token: SmpZgfDwMf91S6V7mYw9gqColimpFb7W56pPCYy9F4' \
  -d '{"reservation_code":"1-801252832144"}'

**响应参数**

```json
{
    "request_id": "RT202507111015382300",
    "error_code": 0,
    "error_msg": "success"
}
```

---

# 渠道

渠道是指如airbnb、美团民宿、携程、Booking等民宿或酒店预订平台。此部分接口用于获取渠道账号和房源相关数据。

---

## 渠道账号列表 /thirdparty/accounts

**请求方式**

GET

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_pZcEW4）

**请求示例**

curl https://api.myhostex.com/thirdparty/accounts?page=1&page_size=1 \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Operator-Id: 10008' \
  -H 'Hostex-Access-Token: SmpZgfDwMf91S6V7mYw9gqColimpFb7W56pPCYy9F4'

**响应示例**

```json
{
    "request_id": "RT1577084949108459",
    "error_code": 0,
    "error_msg": "操作成功",
    "data": {
        "total": 11,
        "list": [
            {
                "id": 101476,
                "thirdparty_type": 0,
                "username": "86-17600142052",
                "created_at": "2021-10-27 13:34:02",
                "last_sync_reservation_time": "2019-12-23 15:09:08",
                "lock_cache_ttl": 0
            }
        ]
    }
}
```

---

## 渠道房源列表 /thirdparty/listings

**请求方式**

GET

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_FmuRAi）

**请求示例**

curl https://api.myhostex.com/thirdparty/listings?thirdparty_account_id=101476 \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Operator-Id: 10008' \
  -H 'Hostex-Access-Token: SmpZgfDwMf91S6V7mYw9gqColimpFb7W56pPCYy9F4'

**响应示例**

```json
{
    "request_id": "RT1635840198819322",
    "error_code": 0,
    "error_msg": "操作成功",
    "data": {
        "list": [
            {
                "origin_house_id": "32812212",
                "thirdparty_type": 1,
                "title": "大床房 ",
                "cover": "{\"id\":null,\"original_url\":\"https:\\/\\/staticfile.tujia.com\\/upload\\/landlordunit\\/day_201103\\/202011031611462176.jpg\"}",
                "in_sale": 1,
                "created_at": "2021-10-26 12:44:35",
                "updated_at": "2021-11-02 14:04:42"
            }
        ]
    }
}
```

---

## 读取账号订单 /thirdparty/sync_reservations

**请求方式**

POST

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_ZC2SL9）

**请求示例**

curl -X POST https://api.myhostex.com/thirdparty/sync_reservations \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Operator-Id: 10008' \
  -H 'Hostex-Access-Token: SmpZgfDwMf91S6V7mYw9gqColimpFb7W56pPCYy9F4' \
  -d '{"thirdparty_account_id":101476}'

**响应参数**

```json
{
    "request_id": "RT1577084949108459",
    "error_code": 0,
    "error_msg": "操作成功"
}
```

---

# 房客消息

## 检索对话 /thread/query

**请求方式**

GET

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_eDj0pb）

**请求示例**

curl https://api.myhostex.com/thread/query?page=1&page_size=1 \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Operator-Id: 10008' \
  -H 'Hostex-Access-Token: SmpZgfDwMf91S6V7mYw9gqColimpFb7W56pPCYy9F4'

**响应示例**

```json
{
    "request_id": "UNKNOWN_ID",
    "error_code": 0,
    "error_msg": "",
    "data": {
        "list": [
            {
                "id": "0-685489979",
                "thirdparty_type": 0,
                "origin_thread_id": "685489979",
                "host_id": "0-159398512",
                "customer_id": "0-260307634",
                "created_at": "2019-05-07 14:58:25",
                "updated_at": "2019-05-07 15:00:34",
                "recent_reservation_code": "0-HMAJ8HR2KZ",
                "recent_thirdparty_house_id": "123456",
                "host": {
                    "id": "0-159398512",
                    "name": "小黑",
                    "full_name": "小黑 韩",
                    "email": "user-sv8t6u97bzi04aq5@guest.airbnb.com",
                    "phone": "+86 186 2961 6237",
                    "photo": "https://z1.muscache.cn/im/pictures/user/e7de023c-2cf3-4ce6-b23d-6bf580d23bf7.jpg?aki_policy=profile_x_medium"
                },
                "customer": {
                    "id": "0-260307634",
                    "name": "丽",
                    "full_name": "丽 谢",
                    "email": "user-5ncghzp65zbygafi@guest.airbnb.com",
                    "phone": "+86 134 7701 9836",
                    "photo": "https://z1.muscache.cn/im/pictures/user/1a2b5b38-fa06-4a7c-a5ee-99ebbcef0c31.jpg?aki_policy=profile_x_medium"
                }
            }
        ],
        "total": 15
    }
}
```

**返回字段说明**

参考对话详情

---

## 对话详情 /thread/detail

**请求方式**

GET

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_tKwxZI）

**请求示例**

curl https://api.myhostex.com/thread/detail?id=6-10011-2830 \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Operator-Id: 10008' \
  -H 'Hostex-Access-Token: SmpZgfDwMf91S6V7mYw9gqColimpFb7W56pPCYy9F4'

**返回字段说明**

**thread:** （详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_XrVZNh）

**host/customer:** （详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_4tNCyj）

---

## 发送消息 inquiry/send

**请求方式**

POST

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_usUN4s）

**请求示例**

curl -X POST https://api.myhostex.com/inquiry/send \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Operator-Id: 10008' \
  -H 'Hostex-Access-Token: SmpZgfDwMf91S6V7mYw9gqColimpFb7W56pPCYy9F4' \
  -d '{"thread_id":"6-10011-2830","message":"hello"}'

**响应参数**

```json
{
    "request_id": "RT15565287970831820",
    "error_code": 0,
    "error_msg": "success"
}
```

---

## 发送推荐房源 inquiry/send_recommend_house

> ⚠️ 仅美团民宿、途家民宿支持发送推荐房源

**请求方式**

POST

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_bOGlBZ）

**请求示例**

curl -X POST https://api.myhostex.com/inquiry/send_recommend_house \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Operator-Id: 10008' \
  -H 'Hostex-Access-Token: SmpZgfDwMf91S6V7mYw9gqColimpFb7W56pPCYy9F4' \
  -d '{"thread_id":"1-10011-2830","origin_house_id":"10291121"}'

**响应参数**

```json
{
    "request_id": "RT15565287970831820",
    "error_code": 0,
    "error_msg": "success"
}
```

---

## 检索消息 /inquiry/query

**请求方式**

GET

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_ZKVatW）

**请求示例**

curl https://api.myhostex.com/inquiry/query?thread_id=1-p2p-qc176216692_1784162-14135556&cursor=0 \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Operator-Id: 10008' \
  -H 'Hostex-Access-Token: SmpZgfDwMf91S6V7mYw9gqColimpFb7W56pPCYy9F4'

**返回字段说明**

list[]: 可参考消息详情接口

next: 下次迭代的 cursor 参数，-1 代表迭代结束。

---

## 消息详情 /inquiry/detail

**请求方式**

GET

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_bByrZz）

**请求示例**

curl https://api.myhostex.com/inquiry/detail?id=1-30cc53636ade76bafc8faa5a8405c5aa-14135556 \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Operator-Id: 10008' \
  -H 'Hostex-Access-Token: SmpZgfDwMf91S6V7mYw9gqColimpFb7W56pPCYy9F4'

**返回字段说明**

**inquiry:** （详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_USOBMm）

**content:** （详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_8Yqwhb）

---

# 事件回调

## 查询回调地址 callback/url_query

**请求方式**

GET

**请求参数**

无

**请求示例**

curl https://api.myhostex.com/callback/url_query \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Access-Token: SmpZgfDwMf91S6V7mYw9gqColimpFb7W56pPCYy9F4'

**响应参数**

```json
{
    "request_id": "RT1563435611912680",
    "error_code": 0,
    "error_msg": "success",
    "data": {
        "url": "http://i.test177.mysite.com/open_api/callback/notify_test",
        "headers": [
            {
                "name": "Authentication",
                "value": "Basic ********"
            }
        ]
    }
}
```

---

## 注册回调地址 callback/register

**请求方式**

POST

**请求参数**

（详见电子表格 EWERsjnqnhNViqtX1wgcvH5tnGf_Oh1Qze）

**请求示例**

curl -X POST https://api.myhostex.com/callback/register \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'Hostex-Access-Token: SmpZgfDwMf91S6V7mYw9gqColimpFb7W56pPCYy9F4' \
  -d '{"notify_url":"http://i.test177.mysite.com/open_api/callback/notify_test","headers":[{"name":"Authentication","value":"Basic ********"}]}'

**响应参数**

```json
{
    "request_id": "RT15565287970831820",
    "error_code": 0,
    "error_msg": "success"
}
```

---

## 消息体说明

当监测到有新渠道消息、渠道订单变化、渠道房态价格变化时，或使用百居易修改房态、房价所产生的同步任务的状态变化时，百居易服务会向合作伙伴注册的url发送http请求。

请求所期望的返回格式：

```json
{
    "status": 0,
    "message": "success"
}
```

> **说明：**
> - status: 0 代表成功，其它值代表失败
> - message: 用于记录失败原因
> - 失败重试次数：3 次
> - 请求超时时间：3 秒

---

### 回调消息类型

| biz_type | 类型说明 |
|----------|----------|
| 1 | 新消息 |
| 2 | 订单变化 |
| 3 | 房间房态房价变化 |
| 4 | 同步任务状态变化 |
| 5 | 渠道房源基础价格变化 |
| 6 | 房型房态房价变化 |

> **注意：** 本文档为百居易（Hostex）开放接口文档，部分内容引用自飞书 Wiki。详细字段说明请参考文档中引用的电子表格 token。
