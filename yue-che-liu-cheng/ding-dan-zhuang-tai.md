---
description: 约车项目中涉及到的前后端状态
---

# 订单状态

约车目前集成的平台为 **首汽** 和 **红旗**

两者目前的后台逻辑和用车流程几乎一致，本文档主要讲前后台下单后订单状态，两平台不同点记录请看[**平台不同点记录**](ping-tai-bu-tong-dian-ji-lu.md)\*\*\*\*

**从用户点击下单开始，向后台下单并开始轮询后台订单状态，App端依据后台状态来刷新前段页面状态，进而刷新页面。后端状态如下：**

```objectivec
/// 后台返回的司机端订单状态
typedef NS_ENUM(NSUInteger, BookCarDriverStatus) {
    
    BookCarDriverStatusBooking = 10,                    ///< 预定中    10
    BookCarDriverStatusAccept = 13,                     ///< 已受理    13
    BookCarDriverStatusBeforeService = 15,              ///< 待服务    15
    BookCarDriverStatusDepart = 20,                     ///< 已出发    20
    BookCarDriverStatusArrive = 25,                     ///< 已到达    25
    BookCarDriverStatusServicing = 30,                  ///< 服务中    30
    BookCarDriverStatusBeforePay = 40,                  ///< 待结算    40
    BookCarDriverStatusPaying = 42,                     ///< 支付中    42
    BookCarDriverStatusDeductMoney = 43,                ///< 扣款中    43
    BookCarDriverStatusPayed = 45,                      ///< 已结算    45
    BookCarDriverStatusComplete = 50,                   ///< 已完成    50
    BookCarDriverStatusCancel = 60,                     ///< 已取消    60
};
```

**前端状态如下**

```objectivec
/// App端订单状态
typedef NS_ENUM(NSUInteger, BookCarStatus) {
    
    BookCarStatusBeforeOrder = 0,       ///< default, 下单前未选择下车地点
    BookCarStatusBeforeOrderConfirm,    ///< 下单前已选下车地点,确认用车
    BookCarStatusOrdered,               ///< 下单后(开始可以有订单操作)
    BookCarStatusCarComing,             ///< 接单后来接人上车
    BookCarStatusCarArrive,             ///< 司机到指定上车地点
    BookCarStatusGetInCar,              ///< 接到客户(不可在进行取消订单操作)
    BookCarStatusComplete,              ///< 订单完成
    
    BookCarStatusCanceled,              ///< 用户取消订单
    BookCarStatusWaitPay,               ///< 待支付
    BookCarStatusWaitappraise,          ///< 待评价
    BookCarStatusHasOpinion,            ///< 客户有异议
};
```

### **下单即开始轮询订单最新状态**

频率10s/次，最多三次请求错误重试，三次后错误直接放弃

**1 轮询订单状态，根据不同状态值处理**

```objectivec
        // 1. 接单后保存司机信息,只要有司机信息，每次都保存，防止直接是恢复状态到30无法展示司机信息
        if (status.integerValue >= BookCarDriverStatusBeforeService) {  // 接单后，有司机信息了,保存司机信息
            
            XYBookCarDriverInfo *driver = [XYBookCarDriverInfo mj_objectWithKeyValues:resultData[@"driverInfo"]];
            
            /// 缓存中有时候拿不到driverInfo，自己存储一份
            [XYBookCarOrderCacheTool saveDriverInfo:driver];
            if (_driverInfo == nil && driver != nil) {
                _driverInfo = driver;
            }
        }
        
        // 2. 接单后开始轮询司机位置
        if (status.integerValue >= BookCarDriverStatusBeforeService && status.integerValue < BookCarDriverStatusPayed) {
            // <接单后，结算前>请求用户车辆位置信息
            [self startPollingDriverLocationWithParam:paramForOrderStatus];
        }
        
        
        // 3. 用户结算或者取消 -- 一到司机确定需支付的订单之后，此处不再轮询，此状态45，直接跳转评价了
        if (status.integerValue < BookCarDriverStatusPayed) {
            // 继续轮询订单状态
            dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(10.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
                [self startPollingOrderStatus];
            });
            
        }
        
        
        // 4.前端状态修改统一在这里面做
        self.bookCarDriverStatus = [status integerValue];
```

**2 根据后端逻辑修改前端页面状态**

```objectivec
/**
 根据不同的后台状态修改对应的App状态和UI
 */
- (void)setBookCarDriverStatus:(BookCarDriverStatus)bookCarDriverStatus
{
    _bookCarDriverStatus = bookCarDriverStatus;
    
    switch (bookCarDriverStatus) {
        case BookCarDriverStatusBooking: //10
        {
            // 用户等待页面UI
            // 1.下单成功，修改当前订单状态
            self.bookCarStatus = BookCarStatusOrdered;
        }
            break;
        case BookCarDriverStatusAccept:
            break;
        case BookCarDriverStatusBeforeService: // 15
        case BookCarDriverStatusDepart: // 20
        {
            // 车辆受理出发  ---> BookCarStatusCarComing
            // 用户接单提示UI
            [self haveCarsAcceptOrder];
        }
            break;
            
        case BookCarDriverStatusArrive: // 25
        {
            // 车辆到达用户起点  ---> BookCarStatusCarArrive
            // 车辆到达提示UI
            [self carDidArriveUserLocation];
        }
            break;
        case BookCarDriverStatusServicing: // 30
        {
            // 车辆接到客户  ---> BookCarStatusGetInCar
            // 车辆到达并开始服务UI
            [self serviceHasStart];
        }
            break;
        case BookCarDriverStatusBeforePay: //40
        {
            // 车辆到达终点等支付 ---> BookCarStatusWaitPay
            // 车辆到达等待支付UI
            [self serviceHasCompleteAndWaitPay];
        }
            break;
        case BookCarDriverStatusPaying:
        case BookCarDriverStatusDeductMoney:
            break;
        case BookCarDriverStatusPayed: // 45
        {
            /// 0. 这里只走一次，防止网络多次返回 45 状态.
            if (self.bookCarStatus == BookCarStatusBeforeOrder) {
                
                // 本函数执行完成后会对状态赋值到起初状态,再次进来就是before状态了，直接返回
                // 可能也是直接恢复状态过来的，此窗台需要重置一下状态
                self.bookCarStatus = BookCarStatusBeforeOrder;
                return;
            }
            
            /// 修改最终结束状态，并跳转对应支付页面
            [self gotoPaymentViewController];
            
            [self serviceHasFinallyComplete];
            
        }
            break;
        case BookCarDriverStatusComplete:
        {
            
            // 订单已经完成
            self.bookCarStatus = BookCarStatusBeforeOrder;
            
            MBProgressHUD * hud = [MBProgressHUD showHUDAddedTo:[UIApplication sharedApplication].keyWindow animated:YES];
            hud.detailsLabelText = @"订单已完成";
            hud.mode = MBProgressHUDModeText;
            [hud hide:YES afterDelay:2.0f];
        }
            break;
        case BookCarDriverStatusCancel:
        {
            // 订单已经取消
            self.bookCarStatus = BookCarStatusBeforeOrder;
            [XYAlertView showAlertTitle:@"提示" message:@"长时间无司机接单，系统自动取消订单" Ok:nil];
        }
            break;
            
        default:
            break;
    }
}
```

\*\*\*\*

