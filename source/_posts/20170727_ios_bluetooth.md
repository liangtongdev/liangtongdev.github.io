---
layout:     post
title:      iOS蓝牙开发 - Central模式
date:       2017-07-27 19:50:12
author:     liangtong
catalog: true
categories: iOS
tags: bluetooth

---






### Central模式   
中心模式(Central Model)，可以简单理解为设备App作为中心，连接其他蓝牙外设设备。区别于外设模式，官方介绍可自行[Google](https://www.google.com/hk)。这里主要介绍iOS上蓝牙设备作为中心模式的集成。

### 功能集成   
  作为蓝牙使用者，程序需要请求蓝牙设备授权，在info.plist配置文件中，添加代码:    
``` Objective-C 
</array>
<key>NSBluetoothPeripheralUsageDescription</key>
<string>蓝牙设备请求描述，比如：程序需要使用蓝牙进行设备发现</string>
```
  项目Targets对应的 *BuildPhases* 中，引入 *CoreBluetooth.framework* ，使用的代码中引入头文件
``` Objective-C 
#import <CoreBluetooth/CoreBluetooth.h>
```

<!-- more -->

#### 请求蓝牙服务
  使用CBCentralManager来进行蓝牙服务管理(授权请求)，监听蓝牙状态变化。    
```ObjectiveC
@property (nonatomic, strong) CBCentralManager *cMgr;

//对象创建成功后，会请求使用蓝牙，CBCentralManagerDelegate协议会回调蓝牙的各个状态
_cMgr = [[CBCentralManager alloc] initWithDelegate:self queue:nil];

#pragma mark - CBCentralManagerDelegate
- (void)centralManagerDidUpdateState:(CBCentralManager *)central{
    switch (central.state) {
        case 0:
            NSLog(@"CBCentralManagerStateUnknown");
            break;
        case 1:
            NSLog(@"CBCentralManagerStateResetting");
            break;
        case 2:
            NSLog(@"CBCentralManagerStateUnsupported");//不支持蓝牙
            break;
        case 3:
            NSLog(@"CBCentralManagerStateUnauthorized");
            break;
        case 4:
            NSLog(@"CBCentralManagerStatePoweredOff");//蓝牙未开启
            break;
        case 5:
            {
                NSLog(@"CBCentralManagerStatePoweredOn");//蓝牙已开启
                // 在中心管理者成功开启后再进行一些操作
                // 搜索外设， 搜索成功之后,会调用我们找到外设的代理方法
                [self.cMgr scanForPeripheralsWithServices:nil // 通过某些服务筛选外设
                                                  options:nil]; // dict,条件
            }
            break;
        default:
            break;
    }
}
```

#### 设备(外设)搜索      
  在设备蓝牙开启的情况下，通过 *CBCentralManager* 的 *scanForPeripheralsWithServices:options:* 方法来搜索蓝牙外设，搜索成功后通过CBCentralManagerDelegate协议回调将外设返回。    
```Objective-C
    //可以通过服务和选项进行蓝牙外设的过滤
    [self.cMgr scanForPeripheralsWithServices:nil  options:nil]

#pragma mark - CBCentralManagerDelegate
//CBCentralManagerDelegate协议对应发现外设，协议回调函数附带外设携带的数据advertisementData和外设的信号强度RSSI
- (void)centralManager:(CBCentralManager *)central didDiscoverPeripheral:(CBPeripheral *)peripheral advertisementData:(NSDictionary<NSString *, id> *)advertisementData RSSI:(NSNumber *)RSSI{
    //此处我们可以将外设设备组织成列表展示出来，以便用户根据实际情况进行特定外设的连接。
    if (![self.self.peripheralList containsObject:peripheral]) {
        [self.peripheralList addObject:item];
        [[NSNotificationCenter defaultCenter] jk_postNotificationOnMainThreadName:BluetoothPeripheralUpdateKey object:nil userInfo:nil];
    }

    //也可以根据信号强度、外设名称或者携带信息，直接过滤并通过CBCentralManager进行连接
    //[self.cMgr connectPeripheral:self.peripheral options:nil];
}
```

#### 设备连接   
  通过CBCentralManager的 *connectPeripheral:options:* 方法进行外设的连接，连接成功后通过CBCentralManagerDelegate协议回调将外设返回。    
```Objective-C
    //外设的连接
    [self.cMgr connectPeripheral:self.peripheral options:nil];

    //中心管理者连接外设成功,协议回调返回
- (void)centralManager:(CBCentralManager *)central didConnectPeripheral:(CBPeripheral *)peripheral{
    //设置外设代理
    peripheral.delegate = self;
}
```   
 此处需要注意的是，蓝牙连接受环境及设备的影响，经常会遇到连接断开的情况。此时可以通过在断开连接的协议回调中重新连接外设即可，此时连接时间非常快。   
```Objective-C
// 丢失连接
- (void)centralManager:(CBCentralManager *)central didDisconnectPeripheral:(CBPeripheral *)peripheral error:(NSError *)error{
    NSLog(@"%s, line = %d, %@=断开连接", __FUNCTION__, __LINE__, peripheral.name);
    if (self.connectedPeripheral) {
        [self.cMgr connectPeripheral:self.peripheral options:nil];
    }
}
```

  通常情况下，我们在外设连接成功的回调中，设置外设回调相关协议代理CBPeripheralDelegate，以便后续的服务发现等操作。

#### 服务发现   
  在蓝牙外设连接成功的情况下，我们通过外设的 *discoverServices:* 方法进行外设服务的发现，发现成功后，通过外设协议CBPeripheralDelegate回调外设提供的服务信息。   
```Objective-C
    //外设服务发现
    [self.peripheral discoverServices:nil];

    //外设服务回调
- (void)peripheral:(CBPeripheral *)peripheral didDiscoverServices:(nullable NSError *)error{
    //    NSLog(@"%s, line = %d", __FUNCTION__, __LINE__);
    for (CBService *s in peripheral.services) {
        NSLog(@" -- 发现服务 -- :%@", s);
    [self.connectedPeripheral discoverCharacteristics:nil forService:s];
    }
}
```   
  通常情况下，我们会对发现的服务进行过滤，查找有用的服务，并查阅其特征值。以上代码读取的所有服务的特征值。

#### 特征值发现    
   外设服务发现的前提下，通过外设的方法 *discoverCharacteristics:forService:* 来进行服务特征值的发现。通过外设协议CBPeripheralDelegate回调服务携带的特征值信息。     
```Objective-C
    //特征值发现
    [self.peripheral discoverCharacteristics:nil forService:s];

    //特征值发现回调
- (void)peripheral:(CBPeripheral *)peripheral didDiscoverCharacteristicsForService:(CBService *)service error:(nullable NSError *)error{
    NSLog(@"%s, line = %d", __FUNCTION__, __LINE__);
    for (CBCharacteristic *characteristics in service.characteristics) {
        //NSLog(@"%s, line = %d, char = %@", __FUNCTION__, __LINE__, cha);
        NSLog(@"characteristic name:%@ value is:%@",characteristics.UUID,characteristics.value);
        [self.peripheral readValueForCharacteristic:characteristics];//读取特征值
        [self.peripheral discoverDescriptorsForCharacteristic:characteristics];//发现特征值所携带的描述信息
    }
}
```    
  发现外设特征值后，我们便可以读取特征值及发现特征值所携带的描述信息了

#### 读取特征值
 发现特征值后，通过外设的方法 *readValueForCharacteristic:* 读取特征值。通过外设协议CBPeripheralDelegate回调服务携带的特征值信息。     
```Objective-C
    //特征值读取
    [self.peripheral readValueForCharacteristic:characteristics];

    //特征值更新回调
- (void)peripheral:(CBPeripheral *)peripheral didUpdateValueForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error{
        NSLog(@"%s, line = %d", __FUNCTION__, __LINE__);
    //特征值更新
}
```    
  特征值携带的UUID、value信息可以直接读取，若特征值发生更新，则通过协议回调进行读取。
#### 读取描述
    发现特征值后，通过外设的方法 *discoverDescriptorsForCharacteristic:* 读取特征值的描述信息。通过外设协议CBPeripheralDelegate回调服务携带的特征值信息。 

    当特征值发生变更的时候，若想要监听，则需要调用蓝牙外设的方法 *- (void)setNotifyValue:(BOOL)enabled forCharacteristic:(CBCharacteristic *)characteristic*
```Objective-C
    //特征值描述字段读取
    [self.peripheral discoverDescriptorsForCharacteristic:characteristics];
    //特征值变更通知
    [self.peripheral setNotifyValue:YES forCharacteristic:characteristics];//接受通知

//特征值描述更新回调
- (void)peripheral:(CBPeripheral *)peripheral didDiscoverDescriptorsForCharacteristic:(CBCharacteristic *)characteristic error:(nullable NSError *)error{
    NSLog(@"===characteristic name:%@",characteristic.service.UUID);
    for (CBDescriptor *d in characteristic.descriptors) {
        NSLog(@"CBDescriptor name is :%@",d.UUID);
    [self.peripheral readValueForDescriptor:d];
    }
}
- (void)peripheral:(CBPeripheral *)peripheral didUpdateValueForDescriptor:(CBDescriptor *)descriptor error:(nullable NSError *)error{
    NSLog(@"Descriptor name:%@ value is:%@",descriptor.characteristic.UUID, descriptor.value);
}

```    
  可读取描述的UUID和value等信息。   
 
### 其他   
​	
