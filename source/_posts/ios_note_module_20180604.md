---
layout:     post
title:      流式布局模块列表
date:       2018-06-04 13:00:00
author:     liangtong
categories: iOS
tags: note

---

几乎每个应用都会有自己的功能模块。整体布局上有的使用抽屉式布局(比如QQ)，有的使用流式布局(比如支付宝)。现在记录下流式布局的功能模块UI界面。

![](/post/note/module_20180604.png)


<!-- more -->

模块展示页面使用 **UICollectionView** 容器，

### 自定义布局

#### 功能模块类别
可以根据需求，此处只展示类别名称
```Objective-C
@interface BModuleCollectionHeaderView : UICollectionReusableView
@property (nonatomic , copy) NSString* typeName;
@end
```
#### 功能模块
展示功能模块图标，名称。及模块类别(普通模式、编辑模式及编辑模式下动作回调)
```Objective-C
typedef NS_ENUM(NSUInteger, BModuleType) {
    BModuleTypeNormal,
    BModuleTypeAdd,
    BModuleTypeRemove,
};
typedef void (^LTxSipprCallbackBlock)(void);

/**
 * 功能模块
 **/
@interface BModuleCollectionViewCell : UICollectionViewCell
@property (nonatomic, strong) NSDictionary* moduleItem;
@property (nonatomic, assign) BModuleType moduleType;
@property (nonatomic, copy) LTxSipprCallbackBlock actionBtnCallback;
@end
```
模块有3种类别，普通类别即模块展示页上使用，点击打开响应的功能模块。**Add、Remove**配合响应的动作 **actionBtnCallback** 在模块管理页面上使用。


### 模块展示页面

注册自定义nib文件（当然如果不使用nib文件的话，此步骤直接跳过）
```Objective-C
    [self.collectionView registerNib:[UINib nibWithNibName:@"BModuleCollectionHeaderView" bundle:nil] forSupplementaryViewOfKind:UICollectionElementKindSectionHeader withReuseIdentifier:BModuleCollectionHeaderViewIdentifier];
    [self.collectionView registerNib:[UINib nibWithNibName:@"BModuleCollectionViewCell" bundle:nil] forCellWithReuseIdentifier:BModuleCollectionViewCellIdentifier];
    self.collectionView.dataSource = self;
    self.collectionView.delegate = self;
    self.collectionView.showsVerticalScrollIndicator = NO;
```

如果对模块界面cell边界进行区分，可能需要有针对性的对**UICollectionView**单元格进行填补。
```Objective-C
- (NSInteger)collectionView:(UICollectionView *)view numberOfItemsInSection:(NSInteger)section{
    NSArray* items = [[self.dataSouce objectAtIndex:section] objectForKey:@"items"];
    NSInteger itemCount = [items count] + 1;
    NSInteger modulo = itemCount % BIM_MODULE_COLLECTION_COLUMNS;//取模
    if (modulo != 0) {
        itemCount += (BIM_MODULE_COLLECTION_COLUMNS - modulo);//填充
    }
    return itemCount;
}
```

展示功能模块类别名称
```Objective-C
- (UICollectionReusableView *)collectionView:(UICollectionView *)collectionView viewForSupplementaryElementOfKind:(NSString *)kind atIndexPath:(NSIndexPath *)indexPath{
    if ([kind isEqualToString: UICollectionElementKindSectionHeader ]){
        BModuleCollectionHeaderView *view =  [collectionView dequeueReusableSupplementaryViewOfKind:kind  withReuseIdentifier:BModuleCollectionHeaderViewIdentifier  forIndexPath:indexPath];
        NSString* typeName = [[self.dataSouce objectAtIndex:indexPath.section] objectForKey:@"type"];
        view.typeName = typeName;
        return view;
    }else{
        return nil;
    }
}
```

功能模块详情
```Objective-C
- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath{
    NSArray* items = [[self.dataSouce objectAtIndex:indexPath.section] objectForKey:@"items"];
    NSDictionary* cellItem;
    NSInteger itemCount = [items count];
    if (indexPath.row < itemCount) {
        cellItem = [items objectAtIndex:indexPath.row];
    }else{
        if (indexPath.section == 0 && indexPath.row == itemCount) {//我的功能中，添加一个 +
            cellItem = @{
                         @"modulename":@"",
                         @"moduletype":@"manage",
                         @"modulecode":@"manage"
                         };
        }else{//补充空闲的Item
            cellItem = @{};
        }
    }
    
    BModuleCollectionViewCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:BModuleCollectionViewCellIdentifier forIndexPath:indexPath];
    cell.moduleType = BModuleTypeNormal;
    cell.moduleItem = cellItem;
    return cell;
}
- (void)collectionView:(UICollectionView *)collectionView didSelectItemAtIndexPath:(NSIndexPath *)indexPath{
    [collectionView deselectItemAtIndexPath:indexPath animated:true];
    UICollectionViewCell* cell = [collectionView cellForItemAtIndexPath:indexPath];
    BModuleCollectionViewCell* moduleCell = (BModuleCollectionViewCell*)cell;
    NSDictionary* item = moduleCell.moduleItem;
    [self showNextWithModuleItem:item];
    
}

-(void)showNextWithModuleItem:(NSDictionary*)moduleItem{
    NSString* moduleType = [moduleItem objectForKey:@"moduletype"];
    if ([moduleType isEqualToString:@"manage"]) {
        [self performSegueWithIdentifier:@"showEdit" sender:nil];
    }else{//hook
    }
}
```

### 模块管理页面
模块管理页面上，如果需要实现拖拽排序，则需要额外给**UICollectionView** 容器添加**长按手势**
```Objective-C
    //此处给其增加长按手势，用此手势触发cell移动效果 - 不直接使用拖拽到原因是因为和列表滑动手势冲突
    UILongPressGestureRecognizer *longGesture = [[UILongPressGestureRecognizer alloc] initWithTarget:self action:@selector(handlelongGesture:)];
    [_collectionView addGestureRecognizer:longGesture];
```

在管理页面上不需要对**UICollectionView**单元格进行填补了。但是cell右上角的 **+**、**-** 角标需要单独配置，此处假定在第一个区域内属于用户订制区域
```Objective-C
- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath;{
    BModuleCollectionViewCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:BModuleCollectionViewCellIdentifier forIndexPath:indexPath];
    NSArray* items = [[self.dataSouce objectAtIndex:indexPath.section] objectForKey:@"items"];
    NSDictionary* cellItem = [items objectAtIndex:indexPath.row];
    cell.moduleItem = cellItem;
    if (indexPath.section == 0) {
        cell.moduleType = BModuleTypeRemove;
    }else{
        cell.moduleType = BModuleTypeAdd;
    }
    __weak __typeof(BModuleCollectionViewCell*)weakCell = cell;
    cell.actionBtnCallback = ^(){//操作元素
        [self editIndexPath:indexPath moduleType:weakCell.moduleType];//此处需要注意
    };
    return cell;
}

// 使用performBatchUpdates:方法，添加移动动画
-(void)editIndexPath:(NSIndexPath*)indexPath moduleType:(BModuleType)moduleType{
    NSIndexPath* fromIndexPath;
    NSIndexPath* toIndexPath;
    
    if (moduleType == BModuleTypeAdd){//添加到我的功能中
        NSMutableArray* sourceArray = [[self.dataSouce objectAtIndex:indexPath.section] objectForKey:@"items"];
        id sourceObj = [sourceArray objectAtIndex:indexPath.row];
        NSMutableArray* destArray = [[self.dataSouce objectAtIndex:0] objectForKey:@"items"];
        if(sourceObj){//将Source插入到Dest中
            [sourceArray removeObject:sourceObj];
            [destArray addObject:sourceObj];
            
            fromIndexPath = indexPath;
            toIndexPath = [NSIndexPath indexPathForRow:destArray.count-1 inSection:0];
        }
    }else if (moduleType == BModuleTypeRemove){//还原到其他类别中
        NSMutableArray* sourceArray = [[self.dataSouce objectAtIndex:0] objectForKey:@"items"];
        NSDictionary* moduleItem = [sourceArray objectAtIndex:indexPath.row];
        NSString* module = [moduleItem objectForKey:@"categoryid"];
        NSMutableArray* destArray;
        NSInteger section = -1;
        for (NSInteger i = 0; i < self.dataSouce.count; ++i) {
            NSDictionary* moduleArrayDic = [self.dataSouce objectAtIndex:i];
            NSString* arrayModule = [moduleArrayDic objectForKey:@"module"];
            if([arrayModule isEqualToString:module]){
                destArray = [moduleArrayDic objectForKey:@"items"];
                section = i;
                break;
            }
        }
        if(destArray){
            [sourceArray removeObject:moduleItem];
            [destArray addObject:moduleItem];
            
            fromIndexPath = indexPath;
            toIndexPath = [NSIndexPath indexPathForRow:destArray.count-1 inSection:section];
        }
    }
    //更新
    [_collectionView performBatchUpdates:^{
        [self.collectionView moveItemAtIndexPath:fromIndexPath toIndexPath:toIndexPath];
    } completion:^(BOOL finished) {
        dispatch_async(dispatch_get_main_queue(), ^{
            [self.collectionView reloadData];
        });
    }];
}
```

允许用户对已经定制的功能模块进行排序
```Objective-C
- (BOOL)collectionView:(UICollectionView *)collectionView canMoveItemAtIndexPath:(NSIndexPath *)indexPath{
    //返回YES允许其item移动
    if (indexPath.section == 0) {
        return YES;
    }else{
        return NO;
    }
}

- (void)collectionView:(UICollectionView *)collectionView moveItemAtIndexPath:(NSIndexPath *)sourceIndexPath toIndexPath:(NSIndexPath*)destinationIndexPath {
    NSMutableArray* sourceArray = [[self.dataSouce objectAtIndex:sourceIndexPath.section] objectForKey:@"items"];
    id sourceObj = [sourceArray objectAtIndex:sourceIndexPath.row];
    NSMutableArray* destArray = [[self.dataSouce objectAtIndex:destinationIndexPath.section] objectForKey:@"items"];
    if(sourceObj){//将Source插入到Dest中
        [sourceArray removeObject:sourceObj];
        [destArray insertObject:sourceObj atIndex:destinationIndexPath.row];
    }
}
#pragma mark 手势操作
- (void)handlelongGesture:(UILongPressGestureRecognizer *)longGesture {
    //判断手势状态
    switch (longGesture.state) {
        case UIGestureRecognizerStateBegan:{//判断手势落点位置是否在路径上
            NSIndexPath *indexPath = [self.collectionView indexPathForItemAtPoint:[longGesture locationInView:self.collectionView]];
            if (indexPath == nil) {
                break;
            }
            //在路径上则开始移动该路径上的cell
            [self.collectionView beginInteractiveMovementForItemAtIndexPath:indexPath];
        }
            break;
        case UIGestureRecognizerStateChanged:{//移动过程当中随时更新cell位置
            CGPoint movedPoint = [longGesture locationInView:self.collectionView];
            NSIndexPath *movedIndexPath = [self.collectionView indexPathForItemAtPoint:movedPoint];
            if (movedIndexPath.section == 0) {//只允许移动已经订阅部分中的功能模块顺序
                [self.collectionView updateInteractiveMovementTargetPosition:movedPoint];
            }
        }
            break;
        case UIGestureRecognizerStateEnded://移动结束后关闭cell移动
            [self.collectionView endInteractiveMovement];
            break;
        default:
            [self.collectionView cancelInteractiveMovement];
            break;
    }
}
```



