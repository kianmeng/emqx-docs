---
# 编写日期
date: 2020-02-20 17:46:13
# 作者 Github 名称
author: wivwiv, terry-xiaoyu
# 关键字
keywords:
# 描述
description:
# 分类
category:
# 引用
ref:
---

# 规则引擎 SQL 语句中可用的字段
SELECT 和 WHERE 子句可用的字段与事件的类型相关。其中 `clientid`, `username` 和 `event` 是通用字段，每种事件类型都有。

## 使用规则引擎 SQL 语句处理消息发布
规则引擎的 SQL 语句可以处理消息发布。 在一个规则语句中，用户可以用 FROM 子句指定一个或者多个主题，当任何消息发布到指定的主题时都会触发该规则。

### FROM 子句可用的事件主题
| 事件主题名                    | 释义     |
| ----------------------------- | :------- |
| $events/message_delivered    | 消息投递 |
| $events/message_acked        | 消息确认 |
| $events/message_dropped      | 消息丢弃 |
| $events/client_connected     | 连接完成 |
| $events/client_disconnected  | 连接断开 |
| $events/client_connack       | 连接确认 |
| $events/client_check_acl_complete | 鉴权结果 |
| $events/session_subscribed   | 订阅     |
| $events/session_unsubscribed | 取消订阅 |

## 普通主题 (消息发布)

当消息在FROM指定的主题上发布时触发规则

|        event        |  事件类型，固定为 "message.publish"   |
| :------------------ | :------------------------------------ |
| id                  | MQTT 消息 ID                          |
| clientid            | 消息来源 Client ID                     |
| username            | 消息来源用户名                          |
| payload             | MQTT 消息体                           |
| peerhost            | 客户端的 IPAddress                    |
| topic               | MQTT 主题                             |
| qos                 | MQTT 消息的 QoS                       |
| flags               | MQTT 消息的 Flags                     |
| headers             | MQTT 消息内部与流程处理相关的额外数据     |
| pub_props           | PUBLISH Properties (仅适用于 MQTT 5.0) |
| timestamp           | 事件触发时间 (ms)                     |
| publish_received_at | PUBLISH 消息到达 Broker 的时间 (ms)   |
| node                | 事件触发所在节点                      |

示例
```sql
SELECT
  payload.msg as msg,
  clientid,
  username,
  payload,
  topic,
  qos
FROM
  "t/#"
```

输出
```json
{
  "username": "u_emqx",
  "topic": "t/a",
  "qos": 1,
  "payload": "{\"msg\":\"hello\"}",
  "msg": "hello",
  "clientid": "c_emqx"
}
```

## 使用规则引擎 SQL 语句处理事件
规则引擎的 SQL 语句既可以处理消息(消息发布)，也可以处理事件(客户端上下线、客户端订阅等)。对于消息，FROM 子句后面直接跟主题名；对于事件，FROM 子句后面跟事件主题。

事件消息的主题以 `"$events/"` 开头，比如 `"$events/client_connected",` `"$events/session_subscribed"。`
如果想让 emqx 将事件消息发布出来，可以在 `emqx_rule_engine.conf` 文件中配置。



### FROM 子句可用的事件主题
| 事件主题名                    | 释义     |
| ----------------------------- | :------- |
| $events/message_delivered    | 消息投递 |
| $events/message_acked        | 消息确认 |
| $events/message_dropped      | 消息丢弃 |
| $events/client_connected     | 连接完成 |
| $events/client_disconnected  | 连接断开 |
| $events/session_subscribed   | 订阅     |
| $events/session_unsubscribed | 取消订阅 |

### $events/message_delivered (消息投递)

当消息被放入底层socket时触发规则

|        字段         |  解释                                 |
| :------------------ | :------------------------------------ |
| id                  | MQTT 消息 ID                         |
| from_clientid       | 消息来源 Client ID                   |
| from_username       | 消息来源用户名                       |
| clientid            | 消息目的 Client ID                   |
| username            | 消息目的用户名                       |
| payload             | MQTT 消息体                          |
| peerhost            | 客户端的 IPAddress                   |
| topic               | MQTT 主题                            |
| qos                 | MQTT 消息的 QoS                      |
| flags               | MQTT 消息的 Flags                    |
| pub_props           | PUBLISH Properties (仅适用于 MQTT 5.0) |
| timestamp           | 事件触发时间 (ms)                    |
| publish_received_at | PUBLISH 消息到达 Broker 的时间 (ms)  |
| node                | 事件触发所在节点                     |

示例
```sql
SELECT
  from_clientid,
  from_username,
  topic,
  qos,
  node,
  timestamp
FROM
  "$events/message_delivered"
```
输出
```json
{
  "topic": "t/a",
  "timestamp": 1645002753259,
  "qos": 1,
  "node": "emqx@127.0.0.1",
  "from_username": "u_emqx_1",
  "from_clientid": "c_emqx_1"
}
```
### $events/message_acked (消息确认)

当消息发送到客户端，并收到客户端回复的ack时触发规则，仅QOS1，QOS2会触发

|        字段         |  解释                                 |
| :------------------ | :------------------------------------ |
| id                  | MQTT 消息 ID                        |
| from_clientid       | 消息来源 Client ID                  |
| from_username       | 消息来源用户名                      |
| clientid            | 消息目的 Client ID                  |
| username            | 消息目的用户名                      |
| payload             | MQTT 消息体                         |
| peerhost            | 客户端的 IPAddress                  |
| topic               | MQTT 主题                           |
| qos                 | MQTT 消息的 QoS                     |
| flags               | MQTT 消息的 Flags                   |
| pub_props           | PUBLISH Properties (仅适用于 MQTT 5.0) |
| puback_props        | PUBACK Properties (仅适用于 MQTT 5.0) |
| timestamp           | 事件触发时间 (ms)                   |
| publish_received_at | PUBLISH 消息到达 Broker 的时间 (ms) |
| node                | 事件触发所在节点                    |

示例
```sql
SELECT
  from_clientid,
  from_username,
  topic,
  qos,
  node,
  timestamp
FROM
  "$events/message_acked"
```
输出
```json
{
  "topic": "t/a",
  "timestamp": 1645002965664,
  "qos": 1,
  "node": "emqx@127.0.0.1",
  "from_username": "u_emqx_1",
  "from_clientid": "c_emqx_1"
}
```

### $events/message_dropped (消息在转发的过程中被丢弃)

当一条消息无任何订阅者时触发规则

|        字段         |  解释                                 |
| :------------------ | :------------------------------------ |
| id                  | MQTT 消息 ID                        |
| reason              | 消息丢弃原因，可能的原因：<br/>no_subscribers: 没有订阅者|
| clientid            | 消息来源 Client ID                  |
| username            | 消息来源用户名                      |
| payload             | MQTT 消息体                         |
| peerhost            | 客户端的 IPAddress                  |
| topic               | MQTT 主题                           |
| qos                 | MQTT 消息的 QoS                     |
| flags               | MQTT 消息的 Flags                   |
| pub_props           | PUBLISH Properties (仅适用于 MQTT 5.0) |
| timestamp           | 事件触发时间 (ms)                   |
| publish_received_at | PUBLISH 消息到达 Broker 的时间 (ms) |
| node                | 事件触发所在节点                    |
示例
```sql
SELECT
  reason,
  topic,
  qos,
  node,
  timestamp
FROM
  "$events/message_dropped"
```
输出
```json
{
  "topic": "t/a",
  "timestamp": 1645003103004,
  "reason": "no_subscribers",
  "qos": 1,
  "node": "emqx@127.0.0.1"
}
```

### $events/delivery_dropped (消息在投递的过程中被丢弃)

当订阅者的消息队列已满时触发规则

|        字段         |  解释                                 |
| :------------------ | :------------------------------------ |
| id                  | MQTT 消息 ID                         |
| reason              | 消息丢弃原因，可能的原因：<br/>queue_full: 消息队列已满(QoS>0)<br/>no_local: 不允许客户端接收自己发布的消息<br/>expired: 消息或者会话过期<br/>qos0_msg: QoS0 的消息因为消息队列已满被丢弃|
| from_clientid       | 消息来源 Client ID                   |
| from_username       | 消息来源用户名                       |
| clientid            | 消息目的 Client ID                   |
| username            | 消息目的用户名                       |
| payload             | MQTT 消息体                          |
| peerhost            | 客户端的 IPAddress                   |
| topic               | MQTT 主题                            |
| qos                 | MQTT 消息的 QoS                      |
| flags               | MQTT 消息的 Flags                    |
| pub_props           | PUBLISH Properties (仅适用于 MQTT 5.0) |
| timestamp           | 事件触发时间 (ms)                    |
| publish_received_at | PUBLISH 消息到达 Broker 的时间 (ms)  |
| node                | 事件触发所在节点                     |

示例
```sql
SELECT
  from_clientid,
  from_username,
  reason,
  topic,
  qos
FROM "$events/delivery_dropped"
```
输出
```json
{
  "topic": "t/a",
  "reason": "queue_full",
  "qos": 1,
  "from_username": "u_emqx_1",
  "from_clientid": "c_emqx_1"
}
```
### $events/client_connected (终端连接成功)

当终端连接成功时触发规则

|        字段         |  解释                                 |
| :------------------ | :------------------------------------ |
| clientid        | 消息目的 Client ID                  |
| username        | 消息目的用户名                      |
| mountpoint      | 主题挂载点(主题前缀)                |
| peername        | 终端的 IPAddress 和 Port            |
| sockname        | emqx 监听的 IPAddress 和 Port       |
| proto_name      | 协议名字                            |
| proto_ver       | 协议版本                            |
| keepalive       | MQTT 保活间隔                       |
| clean_start     | MQTT clean_start                    |
| expiry_interval | MQTT Session 过期时间               |
| is_bridge       | 是否为 MQTT bridge 连接             |
| connected_at    | 终端连接完成时间 (s)                |
| conn_props      | CONNECT Properties (仅适用于 MQTT 5.0) |
| timestamp       | 事件触发时间 (ms)                   |
| node            | 事件触发所在节点                    |

示例
```sql
SELECT
  clientid,
  username,
  keepalive,
  is_bridge
FROM
  "$events/client_connected"
```
输出
```json
{
  "username": "u_emqx",
  "keepalive": 60,
  "is_bridge": false,
  "clientid": "c_emqx"
}
```

### $events/client_disconnected (终端连接断开)

当终端连接断开时触发规则

|        字段         |  解释                                 |
| :------------------ | :------------------------------------ |
| reason          | 终端连接断开原因：<br/>normal：客户端主动断开<br/>kicked：服务端踢出，通过 REST API<br/>keepalive_timeout: keepalive 超时<br/>not_authorized:  认证失败，或者 acl_nomatch = disconnect 时没有权限的 Pub/Sub 会主动断开客户端<br/>tcp_closed: 对端关闭了网络连接<br/>internal_error: 畸形报文或其他未知错误<br/> |
| clientid        | 消息目的 Client ID                                           |
| username        | 消息目的用户名                                               |
| peername        | 终端的 IPAddress 和 Port                                     |
| sockname        | emqx 监听的 IPAddress 和 Port                                |
| disconnected_at | 终端连接断开时间 (s)                                         |
| disconn_props   | DISCONNECT Properties (仅适用于 MQTT 5.0)                    |
| timestamp       | 事件触发时间 (ms)                                            |
| node            | 事件触发所在节点                                             |

示例
```sql
SELECT
  clientid,
  username,
  reason,
  disconnected_at,
  node
FROM
  "$events/client_disconnected"
```
输出
```json
{
  "username": "u_emqx",
  "reason": "normal",
  "node": "emqx@127.0.0.1",
  "disconnected_at": 1645003578536,
  "clientid": "c_emqx"
}
```

<<<<<<< HEAD
## $events/client_connack (连接确认)

当服务端向客户端发送CONNACK报文时触发规则, reason_code 包含各种错误原因代码

| event            | 事件类型，固定为 "client.connack"                |
| ---------------- | :---------------------------------------------- |
| reason_code      | 各种原因代码                                      |
| clientid         | 消息目的 Client ID                               |
| username         | 消息目的用户名                                   |
| peername         | 终端的 IPAddress 和 Port                        |
| sockname         | emqx 监听的 IPAddress 和 Port                   |
| proto_name       | 协议名字                                        |
| proto_ver        | 协议版本                                        |
| keepalive        | MQTT 保活间隔                                   |
| clean_start      | MQTT clean_start                               |
| expiry_interval  | MQTT Session 过期时间                           |
| connected_at     | 终端连接完成时间 (s)                             |
| conn_props       | CONNECT Properties (仅适用于 MQTT 5.0)          |
| timestamp        | 事件触发时间 (ms)                               |
| node             | 事件触发所在节点                                |


MQTT v5.0 协议将返回码重命名为原因码，增加了一个原因码来指示更多类型的错误([Reason code and ACK - MQTT 5.0 new features](https://www.emqx.com/en/blog/mqtt5-new-features-reason-code-and-ack))。
因此reasono_code 在MQTT v3.1.1与MQTT v5.0中有很大的不同。

MQTT v3.1.1
| reason_code                    | 描述      |
| ------------------------------ | --------- |
| connection_accepted            | 已接受连接 |
| unacceptable_protocol_version  | 服务器不支持客户端请求的 MQTT 协议 |
| client_identifier_not_valid    | 客户端 ID 是正确的 UTF-8 字符串，但服务器不允许 |
| server_unavaliable             | 网络连接已建立，但 MQTT 服务不可用 |
| malformed_username_or_password | 用户名或密码中的数据格式错误 |
| unauthorized_client            | 客户端连接未授权 |

MQTT v5.0
| reason_code                   | 描述 |
| ----------------------------- | ----|
| success                       | 连接成功 |
| unspecified_error             | 未指定的错误 |
| malformed_packet              | 畸形数据包 |
| protocol_error                | 协议错误 |
| implementation_specific_error | 实现特定错误 |
| unsupported_protocol_version  | 不支持的协议版本 |
| client_identifier_not_valid   | 客户端标识符无效 |
| bad_username_or_password      | 错误的用户名或密码 |
| not_authorized                | 未经授权 |
| server_unavailable            | 服务器无法使用 |
| server_busy                   | 服务器繁忙 |
| banned                        | 禁止访问 |
| bad_authentication_method     | 错误的身份验证方法 |
| topic_name_invalid            | 主题名称无效 |
| packet_too_large              | 数据包太大 |
| quota_exceeded                | 超出配额 |
| retain_not_supported          | 不支持的retain |
| qos_not_supported             | 不支持的qos |
| use_another_server            | 使用另一台服务器 |
| server_moved                  | 服务器迁移了 |
| connection_rate_exceeded      | 超出连接速率 |

示例
```sql
SELECT
  clientid,
  username,
  reason,
  connected_at,
  node
FROM
  "$events/client_connack"
```
输出
```json
{
  "username": "u_emqx",
  "reason": "success",
  "node": "emqx@127.0.0.1",
  "connected_at": 1645003578536,
  "clientid": "c_emqx"
}
```

## $events/client_check_acl_complete (鉴权结果)

当客户端鉴权结束时触发规则

| event           | 事件类型，固定为 "client.check_acl_complete"                       |
| --------------- | :----------------------------------------------------------- |
| clientid	      | 消息目的 Client ID       |
| username	      | 消息目的用户名           |
| peerhost	      | 客户端的 IPAddress       |
| topic	          | MQTT 主题               |
| action	      | publish or subscribe, 发布或者订阅事件 |
| result          | allow or deny，鉴权结果            |
| is_cache        | true or false，鉴权时数据的来源 <br/>is_cache为true时，鉴权数据来源于cache <br/>is_cache为false时，鉴权数据来源于插件           |
| timestamp	      | 事件触发时间 (ms)       |
| node	          | 事件触发所在节点        |

示例
```sql
SELECT
  clientid,
  username,
  topic,
  action,
  result,
  is_cache,
  node
FROM
  "$events/client_check_acl_complete"
```
输出
```json
{
  "username": "u_emqx",
  "topic": "t/a",
  "action": "publish",
  "result": "allow",
  "is_cache": "false",
  "node": "emqx@127.0.0.1",
  "clientid": "c_emqx"
}
```

## $events/session_subscribed (终端订阅成功)
=======
### $events/session_subscribed (终端订阅成功)
>>>>>>> origin/release-4.3

当终端订阅成功时触发规则

|        字段         |  解释                       |
| :------------------ | :--------------------------|
| clientid  | 消息目的 Client ID                    |
| username  | 消息目的用户名                        |
| peerhost  | 客户端的 IPAddress                    |
| topic     | MQTT 主题                             |
| qos       | MQTT 消息的 QoS                       |
| sub_props | SUBSCRIBE Properties (仅适用于 5.0)  |
| timestamp | 事件触发时间 (ms)                     |
| node      | 事件触发所在节点                      |

示例
```sql
SELECT
  clientid,
  username,
  topic,
  qos
FROM
  "$events/session_subscribed"
```
输出
```json
{
  "username": "u_emqx",
  "topic": "t/a",
  "qos": 1,
  "clientid": "c_emqx"
}
```

### $events/session_unsubscribed (取消终端订阅成功)

当取消终端订阅成功时触发规则

|        字段         |  解释                         |
| :------------------ | :---------------------------- |
| clientid  | 消息目的 Client ID                      |
| username  | 消息目的用户名                          |
| peerhost  | 客户端的 IPAddress                      |
| topic     | MQTT 主题                               |
| qos       | MQTT 消息的 QoS                         |
| unsub_props | UNSUBSCRIBE Properties (仅适用于 5.0)  |
| timestamp | 事件触发时间 (ms)                       |
| node      | 事件触发所在节点         

示例
```sql
SELECT
  clientid,
  username,
  topic,
  qos
FROM
  "$events/session_unsubscribed"
```
输出
```json
{
  "username": "u_emqx",
  "topic": "t/a",
  "qos": 1,
  "clientid": "c_emqx"
}
```

[下一部分，规则引擎内置函数](rule-engine_buildin_function.md)
