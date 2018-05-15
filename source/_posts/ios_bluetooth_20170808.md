---
layout:     post
title:      iOS蓝牙开发 - Peripheral模式
date:       2017-08-08 18:50:12
author:     liangtong
catalog: true
categories: iOS
tags: bluetooth

---






### Peripheral模式   
周边模式(Peripheral Model)，可以简单理解为设备App作为周边，供其他蓝牙设备连接。区别于中心模式。
蓝牙设备不能同时作为周边和中心设备，在某次连接中，只能担当一个角色。


<!-- more -->

### 功能集成   
  作为蓝牙使用者，程序需要请求蓝牙设备授权，在info.plist配置文件中，添加代码:    
``` Objective-C 
</array>
<key>NSBluetoothPeripheralUsageDescription</key>
<string>蓝牙设备请求描述，比如：程序需要使用蓝牙进行设备发现</string>
```
  项目Targets对应的 *Build Phases* 中，引入 *CoreBluetooth.framework* ，使用的代码中引入头文件
``` Objective-C 
#import <CoreBluetooth/CoreBluetooth.h>
```

#### 请求蓝牙服务
  使用CBPeripheralManagerDelegate来进行蓝牙服务管理(授权请求)，监听蓝牙状态变化。    
```ObjectiveC
@property (nonatomic, strong) CBPeripheralManagerDelegate *perManager;

//对象创建成功后，会请求使用蓝牙，CBPeripheralManagerDelegate协议会回调蓝牙的各个状态
_perManager = [[CBPeripheralManager alloc] initWithDelegate:self queue:nil];

#pragma mark - CBPeripheralManagerDelegate
- (void)peripheralManagerDidUpdateState:(CBPeripheralManager *)peripheral{
    if (peripheral.state == CBManagerStatePoweredOn) {
        
    }
}
```
在蓝牙开启的情况下，可以创建服务、创建特质值等。



#### 带特质值的服务创建     
  在设备蓝牙开启的情况下，可以建立服务 *CBMutableService* 和特征值 *CBCharacteristic* 并通过 * CBPeripheralManagerDelegate* 的 *addService:* 方法将带有特征值的服务注册给蓝牙设备。注册成功后，通过 * CBPeripheralManagerDelegate* 协议将服务返回。    
```Objective-C
    //创建服务，其中服务关键字UUID换成自己的
    CBMutableService * service = [[CBMutableService alloc] initWithType:
                                  [CBUUID UUIDWithString:@"service UUID"] primary:YES];
    //创建特征值，其中特征值的UUID换成自己的，特征值可以携带value信息（nsdata）
    CBCharacteristic * characteristic = [[CBMutableCharacteristic alloc] initWithType:[CBUUID UUIDWithString:@"characteristic UUID"]
                                                                           properties:(CBCharacteristicPropertyRead |
                                                                                        CBCharacteristicPropertyWrite |
                                                                                        CBCharacteristicPropertyNotify) 
                                                                                value:[@"characteristic value"
                                                                    dataUsingEncoding:NSUTF8StringEncoding]  
                                                                          permissions:CBAttributePermissionsReadable];

    //将特征值加入到服务中
    service.characteristics = @[characteristic];

    //给设备添加服务，注册服务
    [_perManager addService:service];

#pragma mark - CBPeripheralManagerDelegate
//服务注册成功后，会通过以下回调将信息返回
- (void)peripheralManager:(CBPeripheralManager *)peripheral 
  			didAddService:(CBService *)service
              		error:(NSError *)error{
    //对于注册成功的服务，蓝牙设备可以直接进行广播
    //开启广播，此处我们广播刚才注册的服务，UUID是：service uuid
    [_perManager startAdvertising:
     @{CBAdvertisementDataServiceUUIDsKey:[CBUUID UUIDWithString:@"service UUID"]}];	
}
```

注：一个服务可以携带多个特征值，一个蓝牙周边设备可以注册多个服务。

#### 开启广播   
  通过CBPeripheralManager的 *startAdvertising:* 方法开启广播，成功后通过CBPeripheralManagerDelegate协议回调。    
```Objective-C
    //开启广播
    [_perManager startAdvertising:@{CBAdvertisementDataServiceUUIDsKey:[CBUUID UUIDWithString:@"service UUID"]}];

    //开启广播的回调
- (void)peripheralManagerDidStartAdvertising:(CBPeripheralManager *)peripheral 
  										error:(NSError *)error {
}
```

#### 数据读写   
 
当外设蓝牙设备收到ATT特征值请求时，协议 * CBPeripheralManagerDelegate* 的方法会被触发。参照协议描述，在处理数据读／写操作的协议回调方法中，必须执行方法 *respondToRequest:withResult:*

```Objective-C
// 读数据请求
- (void)peripheralManager:(CBPeripheralManager *)peripheral didReceiveReadRequest:(CBATTRequest *)request {
    //请求的响应
    NSData *data = request.characteristic.value; //蓝牙中心设备想读取的数据
    [request setValue:data];// 读取的值
    [_perManager respondToRequest:request withResult:CBATTErrorSuccess];
}

//写数据请求
- (void)peripheralManager:(CBPeripheralManager *)peripheral didReceiveWriteRequests:(NSArray<CBATTRequest *> *)requests {
    //请求的响应
    //需要转换成CBMutableCharacteristic对象才能进行写值
    for (CBATTRequest* request in requests) {
        CBMutableCharacteristic *c =(CBMutableCharacteristic *)request.characteristic;
        c.value = request.value;
        [_perManager respondToRequest:[requests firstObject] withResult:CBATTErrorSuccess];
    }
}
```
​
#### 特征值订阅
当蓝牙中心设备在进行蓝牙设备服务的特征发现时，若将特征值配置包含被通知 *CBCharacteristicPropertyNotify* ， 则协议中的以下回调会被调用。

```Objective-C
//订阅characteristics
-(void)peripheralManager:(CBPeripheralManager *)peripheral central:(CBCentral *)central didSubscribeToCharacteristic:(CBCharacteristic *)characteristic{
    NSLog(@"订阅了 %@的数据",characteristic.UUID);
    
    //每秒执行一次给主设备发送一个当前时间的秒数
    _timer = [NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(sendData:) userInfo:characteristic  repeats:YES];
}
//如果不设置订阅的设备，则代表全部。
-(void)sendData:(NSTimer *)t {
    CBMutableCharacteristic *characteristic = t.userInfo;
    [_perManager updateValue:[@"Subscribe To Characteristic Value Send " dataUsingEncoding:NSUTF8StringEncoding] forCharacteristic:characteristic onSubscribedCentrals:nil];
}

//取消订阅characteristics
-(void)peripheralManager:(CBPeripheralManager *)peripheral central:(CBCentral *)central didUnsubscribeFromCharacteristic:(CBCharacteristic *)characteristic{
    NSLog(@"取消订阅 %@的数据",characteristic.UUID);
    
    //取消回应
    [_timer invalidate];
}
```
	
