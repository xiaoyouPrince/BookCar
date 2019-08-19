---
description: 只有用户当前在约车主页的时候可以轮询当前订单状态
---

# 订单状态轮询逻辑

### 何时需要轮询订单状态

只有当用户当前在约车主页且存在未完成订单的情况才需要轮询订单，以确保App展示正确的约车状态页面。

约车主页 XYBookCarViewController 中提供一个展示是否需要轮询订单状态的接口变量

```objectivec
@property(nonatomic , assign)     BOOL canPollingOrderStatus;
```

{% hint style="info" %}
**是否可以轮询订单**

 性能优化，当用户退出约车页面，停止轮询订单（ARC下退出页面，self并未被直接销毁，加之约车主页逻辑过多，难以保证不出现循环引用，遂加此字段手动判断）

 **@note** 只有在当前主页才可以轮询订单状态
{% endhint %}

实际就是当XYBookCarVC处于当前页面的时候才可以轮询订单状态，非当前页面停止轮询。此要配合订单恢复一起使用。因为退出约车主页，不会再轮询，当再次进入页面，也要查看是否要进行订单恢复。有需要恢复订单，会重新轮询，否则即不用恢复亦无须再次轮询

```objectivec
ViewWillAppear 方法中设置为 canPollingOrderStatus = YES

viewDidDisappear 设置为 canPollingOrderStatus = NO

```



