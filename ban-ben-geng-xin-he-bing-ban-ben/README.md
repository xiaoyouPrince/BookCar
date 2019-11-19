---
description: 此版本合并首汽约车 && 红旗智行两个平台，使得用户页面完全统一，App处理不同平台间差异，方便用户使用！
---

# 版本更新\(合并版本\)

本版本主要修改点记录如下

* [约车页面入口统一为企业出行，不再进行各平台差异化初始化](yue-che-gong-neng-ye-mian-tong-yi-ru-kou.md)
* [约车平台和用户类型工具优化，可处理更多平台且兼容旧版单平台处理](yue-che-ping-tai-he-yong-hu-lei-xing-gong-ju.md)
* [下单接口](xia-dan-jie-kou-he-bing-ban.md)
* [订单轮询](ding-dan-lun-xun-he-bing-ban.md)
* [订单恢复](ding-dan-hui-fu-he-bing-ban.md)

### 已知存在的问题

* 订单恢复：订单恢复有个 10s 轮询问题，如果10s 内频繁进入约车页面，会造成，每次进入调用一次查询有没有未完成订单接口，每次进入都会重启10s 轮询机制，导致接口调用过于频繁，这是一个**设计bug**



