---
layout:     post
title:      蓝牙Central模式 - 水准仪蓝牙交互
date:       2017-08-24 18:50:12
author:     liangtong
catalog: true
categories: iOS
tags: bluetooth

---


​	工程测量中，[Trimble](http://www.trimble.com/)电子水准仪(DiNi Level)在测量完成后，通过蓝牙模块与外设进行数据交互，本文介绍外设如何作为中心模式与水准仪进行数据交互。

#### 前期准备

前期准备工作包括以下几个方面：   
 + 蓝牙外设扫描
 + 蓝牙外设连接
 + 外设服务发现
 + 特征值发现
   + 订阅


<!-- more -->

可以参照[iOS蓝牙开发 - Central模式](https://l900416.github.io/2017/07/27/bluetooth_central/)，此处不再赘述，参照以下摘要信息：

```bash
2017-08-24 16:15:39.860 RoadBridgeOnline[1067:189910] CBCentralManagerStatePoweredOn
2017-08-24 16:15:50.121 RoadBridgeOnline[1067:189910] -[BluetoothViewModel centralManager:didConnectPeripheral:], line = 198, CRTB_BT_MODULE=连接成功
2017-08-24 16:15:50.188 RoadBridgeOnline[1067:189910]  -- 发现服务 -- :<CBService: 0x15e6301c0, isPrimary = YES, UUID = 1101>
2017-08-24 16:15:50.189 RoadBridgeOnline[1067:189910]  -- 发现服务 -- :<CBService: 0x15e6a6590, isPrimary = YES, UUID = 00001016-D102-11E1-9B23-00025B00A5A5>
2017-08-24 16:15:50.191 RoadBridgeOnline[1067:189910] -[BluetoothViewModel peripheral:didDiscoverCharacteristicsForService:error:], line = 236
2017-08-24 16:15:50.191 RoadBridgeOnline[1067:189910] characteristic name:1102 value is: 

2017-08-24 16:15:50.193 RoadBridgeOnline[1067:189910] characteristic name:1103 value is:
2017-08-24 16:15:50.194 RoadBridgeOnline[1067:189910] -[BluetoothViewModel peripheral:didDiscoverCharacteristicsForService:error:], line = 236
2017-08-24 16:15:50.195 RoadBridgeOnline[1067:189910] characteristic name:00001013-D102-11E1-9B23-00025B00A5A5 value is:
2017-08-24 16:15:50.197 RoadBridgeOnline[1067:189910] characteristic name:00001018-D102-11E1-9B23-00025B00A5A5 value is:
2017-08-24 16:15:50.198 RoadBridgeOnline[1067:189910] characteristic name:00001014-D102-11E1-9B23-00025B00A5A5 value is:
2017-08-24 16:15:50.199 RoadBridgeOnline[1067:189910] characteristic name:00001011-D102-11E1-9B23-00025B00A5A5 value is:
2017-08-24 16:15:50.248 RoadBridgeOnline[1067:189910] -[BluetoothViewModel peripheral:didUpdateValueForCharacteristic:error:], line = 248
2017-08-24 16:15:50.248 RoadBridgeOnline[1067:189910] characteristic name:1102 value is: 

```


#### 特征值订阅／监听

水准仪与蓝牙外设的通讯，主要是通过特征值来实现，故此处进行单独介绍。在特征值发现之后，我们可以根据特征值进行过滤，监听特定的特征值，这样，当特征值发生变更的时候，会通过CBPeripheralDelegate协议回调特征值数据。使用蓝牙外设对象 *CBPeripheral* 的方法 *setNotifyValue:forCharacteristic:* 来进行监听

```Objective-C
	//订阅 - 接受通知
    [self.connectedPeripheral setNotifyValue:YES forCharacteristic:characteristics];
```

通过协议回调处理特征值。

```Objective-C
/**
 * CBPeripheralDelegate协议，当特征值发生变更的时候，回调
 **/
- (void)peripheral:(CBPeripheral *)peripheral didUpdateValueForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error{
        NSLog(@"%s, line = %d", __FUNCTION__, __LINE__);
    NSString* valueString = [[NSString alloc] initWithData:characteristic.value  encoding:NSUTF8StringEncoding];
    [self.readValueArray addObject:valueString];
    [self postRecievedCharacteristicValue];
    
    NSLog(@"characteristic name:%@ value is:%@",characteristic.UUID,valueString);
//    NSLog(@"接收到的数组：\n%@",self.readValueArray);
}

/**
  * 将水准仪返回的数据进行处理，获取关键字段，数据处理成功后，直接发全局Event通知。
  * 单点测量
  * 水准线路测量 - 后视
  * 水准线路测量 - 前视
  **/
-(void)postRecievedCharacteristicValue{
    if ([self.readValueArray count] > 0) {
        NSString * fullString = [self.readValueArray componentsJoinedByString:@""];
        NSLog(@"待处理的特质值为：\n %@ \n",fullString);
        
        NSArray* sepArray = [fullString componentsSeparatedByString:@"|"];
        NSMutableDictionary* postDic = [[NSMutableDictionary alloc] init];
        
	    //处理字符串数据，将不同数据进行封装，代码略
        
        if ([[postDic allKeys] count] > 0) {//直接全局通知
            [[NSNotificationCenter defaultCenter] jk_postNotificationOnMainThreadName:RBBluetoothMeasureUpdateKey object:postDic];
        }
    }
}
```
需要注意的是水准仪蓝牙数据的回传时将字符串进行了分隔。我们需要首先进行数据拼接，之后才能进行数据提取操作，以下是拼接后的字符串及原始字符串：

```Objective-C
//以下是拼接后的字符串（水准线路测量 - 前视）
For M5|Adr    32|KD1      5-a    g 15:49:151  L1|Rf        1.49585 m   |HD          9.988 m   |                      | 
For M5|Adr    33|KD1      5-a    g 15:49:15   L1|                      |                      |Z        -0.43374 m   | 

//以下是回调log信息：
For M5|Adr      |K
2017-08-24 16:18:42.299 RoadBridgeOnline[1067:189910] -[BluetoothViewModel peripheral:didUpdateValueForCharacteristic:error:], line = 248
2017-08-24 16:18:42.300 RoadBridgeOnline[1067:189910] characteristic name:1102 value is:D1       16    2 16:
2017-08-24 16:18:42.327 RoadBridgeOnline[1067:189910] -[BluetoothViewModel peripheral:didUpdateValueForCharacteristic:error:], line = 248
2017-08-24 16:18:42.328 RoadBridgeOnline[1067:189910] characteristic name:1102 value is:17:37  L41|         
2017-08-24 16:18:42.329 RoadBridgeOnline[1067:189910] -[BluetoothViewModel peripheral:didUpdateValueForCharacteristic:error:], line = 248
2017-08-24 16:18:42.330 RoadBridgeOnline[1067:189910] characteristic name:1102 value is:             |      
2017-08-24 16:18:42.357 RoadBridgeOnline[1067:189910] -[BluetoothViewModel peripheral:didUpdateValueForCharacteristic:error:], line = 248
2017-08-24 16:18:42.358 RoadBridgeOnline[1067:189910] characteristic name:1102 value is:                |Z  
2017-08-24 16:18:42.387 RoadBridgeOnline[1067:189910] -[BluetoothViewModel peripheral:didUpdateValueForCharacteristic:error:], line = 248
2017-08-24 16:18:42.398 RoadBridgeOnline[1067:189910] characteristic name:1102 value is:      44.98000 m   |
2017-08-24 16:18:42.477 RoadBridgeOnline[1067:189910] -[BluetoothViewModel peripheral:didUpdateValueForCharacteristic:error:], line = 248
```

#### 测量数据处理

在前台界面相关的 *界面控制器* 中，使用 *NSNotificationCenter* 进行监听，对监听到的数据进行相应的业务逻辑处理

```Objective-C
-(void)viewWillAppear:(BOOL)animated{
    [super viewWillAppear:animated];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(recieveLocalNotification:) name:BluetoothPeripheralUpdateKey object:nil];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(recieveLocalNotification:) name:RBBluetoothMeasureUpdateKey object:nil];
}

-(void)recieveLocalNotification:(NSNotification*)notification{
    NSString* notificationKeyName = [notification name];
    //蓝牙连接状态发生变更
    //蓝牙单点测量数据变更通知
    if ([notificationKeyName isEqualToString:BluetoothPeripheralUpdateKey]) {
        [self updateBluetoothConnectionImage];
    }else if ([notificationKeyName isEqualToString:RBBluetoothMeasureUpdateKey]) {
        NSDictionary* measuredDic = notification.object;
        [self uploadBluetoothMeasureData:measuredDic];
    }
}
-(void)viewWillDisappear:(BOOL)animated{
    [super viewWillDisappear:animated];
    [[NSNotificationCenter defaultCenter] removeObserver:self];
}
```
​	
