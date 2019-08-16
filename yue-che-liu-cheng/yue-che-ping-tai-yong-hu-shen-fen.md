# 约车平台 && 用户身份

### 当前约车功能涉及的平台和用户身份:

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x5E73;&#x53F0;</th>
      <th style="text-align:left">&#x7528;&#x6237;&#x8EAB;&#x4EFD;&#x7C7B;&#x578B;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">&#x9996;&#x6C7D;&#x7EA6;&#x8F66;</td>
      <td style="text-align:left">
        <p>1.&#x666E;&#x901A;&#x96C7;&#x5458;&#x7528;&#x6237;</p>
        <p>2.OA&#x7528;&#x6237;</p>
        <p>3.OA&#x767D;&#x540D;&#x5355;&#x7528;&#x6237;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7EA2;&#x65D7;&#x667A;&#x884C;</td>
      <td style="text-align:left">1.&#x666E;&#x901A;&#x96C7;&#x5458;&#x7528;&#x6237;</td>
    </tr>
  </tbody>
</table>### **约车平台代码定义**

```text
/// 约车平台，默认为首汽
typedef NS_ENUM(NSUInteger, BookCarPlatform) {
    BookCarPlatformShouQi,
    BookCarPlatformHongQi
};
```

### **用户身份类型定义**

```text
/// 首汽用户类型
typedef enum : NSUInteger {
    UserTypeDefault = 1,            // 普通雇员
    UserTypeFescoOA = 2,            // OA 用户
    UserTypeFescoOALeader = 3       // OA 用户领导(白名单)
} UserType;

/// 红旗用户类型
typedef enum : NSUInteger {
    HongQiUserTypeDefault = 1,            // 普通雇员
} HongQiUserType;

/// Channel-Type验证请求的标识 1=雇员 2=OA
```

管理工具为  **XYBookCarPlatformAndUserTypeTool** 提供方法为设置平台和设置用户身份。用户平台和不同身份最终以网络请求头字段的形式传递到后台。

> 平台请求头字段【key-value】
>
> **Channel-Type**    1=雇员 2=OA

> 用户身份请求头字段【key-value】
>
> **Transport-Channels**    “sqyc”=首汽约车 "hqzx"=红旗智行

工具类提供了主要的设置平台和用户身份类型的方法，并抽取成对应的宏，方便使用，工具类的使用基础方法为 设置平台和设置用户身份。

{% hint style="info" %}
**工具类存储平台和用户类型思路**

内部维护一个保存到userdefault的二维字典。



### **输入**

1. 设置平台  **一切基于平台类型**
2. 设置用户类型 内部会维护一个存储平台和用户类型的二级字典，每次有对应的值变动会修改对应的字典存储值



### **输出**

1. 查看当前平台
2. 查看当前用户类型\(基于当前平台\)
{% endhint %}

### 





![&#x54C8;&#x54C8;&#x54C8;](../.gitbook/assets/logo.png)

### 以下为工具类头文件接口声明

```text
@interface XYBookCarPlatformAndUserTypeTool : NSObject

#pragma mark - 输入设置 平台和身份类型

/// 设置平台。后续方法均基于平台，默认首汽平台
+ (void)setPlatform:(BookCarPlatform)plat;

/// 仅设置用户类型，平台默认首汽。只有首汽有多用户身份
+ (void)setUserType:(ShouQiUserType)userType;

#pragma mark - 获取 平台和身份类型

/// 获取完整的当前平台和用户身份信息
+ (XYBookCarPlatformAndUserType *)currentPlatformAndUserType;

/*获取对应平台下的用户身份信息*/
+ (UserType)getUserTypeForPlatfrom:(BookCarPlatform)plat;

/**获取当前用车平台下的用户身份类型*/
+ (UserType)getCurrentUserType;

/**获取当前用车平台下的用户身份类型String*/
+ (NSString *)getCurrentUserTypeString;


/// 获取当前平台的key,sqyc hqzx
+ (NSString *)getCurrentPlatformKey;

/// 获取当前平台的title,约车页面的title可由此获得
+ (NSString *)getCurrentPlatformTitle;

/// 平台是否为首汽约车
+ (BOOL)isPlatformShouQi;

/// 当前用户类型是否为OA，只存在于首汽约车平台
+ (BOOL)isUserTypeOA;
@end
```

### 抽取常用宏

```text
// 几个常用宏

/// 设置用户类型，@"0" @"1" @"2"
#define kBookCarSetSQUserType(type) 

/// 获取约车首汽用户类型
#define kBookCarGetSQUserType 

/// 获取是否为OA用户，只有首汽用户类型存在OA用户
#define kBookCarUserTypeOA 

/// 获取当前约车平台
#define kBookCarType 

/// 判断当前约车平台是否为红旗智行
#define kBookCarTypeHQZX 

/// 判断当前约车平台是否为首汽约车
#define kBookCarTypeSQYC 
```

