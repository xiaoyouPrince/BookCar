---
description: 合并版订单恢复逻辑基本同单平台板订单恢复逻辑，本文主要记录了合并版的改动点
---

# 订单恢复\_\(合并版\)

可以先参考一下分开版的[订单恢复逻辑](../yue-che-liu-cheng/ding-dan-hui-fu-luo-ji.md)

合并版主要区别如下：

1. 接口地址改为 [getOrderDescV1](http://47.93.126.37/car/izu/order/getOrderDescV1)，参数为空
2. 首次进入为平台为“sqhq”，用户类型为用户所选，【请求头】
3. 请求到当前列表
4. 订单 status &lt; 15 即未接单，此时直接重新轮询订单即可，
5. 订单 status &gt; 15 即已接单，此时归档订单平台，用户类型，和订单相关信息，--进行订单恢复，

