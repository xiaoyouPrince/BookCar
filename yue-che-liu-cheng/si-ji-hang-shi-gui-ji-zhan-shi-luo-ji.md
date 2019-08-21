---
description: 司机行驶轨迹逻辑和对应时段大头针展示信息逻辑
---

# 司机行驶轨迹展示逻辑

## 绘制司机行驶轨迹司机

司机轨迹绘制主要分两种情况

1. 接单后，乘客上车前 --&gt; 司机来接乘客
2. 乘客上车后，去往目的地 

{% hint style="info" %}
 两个平台逻辑相同，但是红旗平台接口缺少，司机预估时间和距离字段，这里的展示相应也就不同
{% endhint %}

对应展示逻辑如图所示

![&#x7ED8;&#x5236;&#x53F8;&#x673A;&#x8F68;&#x8FF9;&#x548C;&#x5BF9;&#x5E94;&#x72B6;&#x6001;&#x4E0B;&#x63D0;&#x793A;&#x4FE1;&#x606F;](../.gitbook/assets/ping-mu-kuai-zhao-20190821-shang-wu-10.39.40.png)

### 地图SDK 和 对应的路径绘制

涉及到的地图SDK ，具体版本忘记了，以后更新的时候可以统一更新

1. AMapSearchKit.framework【用于POI，查询地图周围信息，编码等】
2. MAMapKit.framework【地图展示相关】
3. AMapFoundationKit.framework【地图框架基础】
4. AMapLocationKit.framework【定位相关】
5. AMapNaviKit.framework【导航相关，导航路径绘制】
6. AMapNavi.bundle【资源】
7. AMap.bundle【资源】



