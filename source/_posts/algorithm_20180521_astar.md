---
layout:     post
title:      A*算法
date:       2018-05-21 14:25:19
author:     liangtong
categories: 算法
tags: 算路

---

本文分享简单A*算法的实现。

![](/post/note/algorithm_astar_20180521.png)


<!-- more -->

### 声明

  A* 算法结点，包含当前节点编码，路径节点以及各种估值代价。并不是A*算法导航结点(包含通达关系的结点)

```Objective-C
/**
 * A*算法结点
 **/
@interface AStarNode : NSObject
@property (nonatomic, copy) NSString *nodeId;//待计算的导航结点
@property (nonatomic, strong) AStarNode *preNode;//存储导航父节点，用于向上寻找路径
@property (nonatomic, assign) double fn;//估价函数的值。getter{ gn + hn}
@property (nonatomic, assign) double gn;//从起点到当前点的实际代价
@property (nonatomic, assign) double hn;//从当前点到结点的估算代价

+(instancetype)instanceWithNodeId:(NSString*)nodeId preNode:(AStarNode*)preNode gn:(double)gn hn:(double)hn;
@end
```


  A* 算法。

```Objective-C
@interface ALEAStar : NSObject
/**
 * A* 算法
 *
 * @param fromId 起始位置编码
 * @param toId 终点位置编码
 * @param nodeDic 图节点(包含通达关系)字典{节点编码：节点}
 *
 @return 规划的路径；若为nil，则表示算路失败
 **/
+(NSMutableArray*)aStarPathFrom:(NSString*)fromId to:(NSString*)toId nodes:(NSMutableDictionary*)nodeDic;
@end
```


### 实现

  结点的实现比较简单。

```Objective-C
/**
 * A*算法节点
 **/
@implementation AStarNode
+(instancetype)instanceWithNodeId:(NSString*)nodeId preNode:(AStarNode*)preNode gn:(double)gn hn:(double)hn{
    AStarNode* retInstance = [[AStarNode alloc] init];
    retInstance.nodeId = nodeId;
    retInstance.preNode = preNode;
    retInstance.gn = gn;
    retInstance.hn = hn;
    return retInstance;
}
-(double)fn{
    return  _gn + _hn;
}
@end
```

  算法的实现。需要注意的是估算函数的使用及各A*结点的gn的更新。

```Objecitve-C
/**
 * A* 算法
 *
 * @param fromId 起始位置编码
 * @param toId 终点位置编码
 * @param nodeDic 图节点(包含通达关系)字典{节点编码：节点}
 *
 @return 规划的路径；若为nil，则表示算路失败
 **/
+(NSMutableArray*)aStarPathFrom:(NSString*)fromId to:(NSString*)toId nodes:(NSMutableDictionary*)nodeDic{
    //数据有效性检查
    if(!fromId || !toId || [fromId isEqualToString:toId]){
        NSLog(@"起点(或终点)不符合A*算法要求");
        return nil;
    }
    if ([[nodeDic allKeys] count] == 0) {
        NSLog(@"A*算法节点列表为空，不符合算法要求");
        return nil;
    }
    
    //构造open表，close表
    NSMutableArray* open = [[NSMutableArray alloc] init];
    NSMutableArray* close = [[NSMutableArray alloc] init];
    ALENavigationNode* sNode = [nodeDic objectForKey:fromId];
    ALENavigationNode* eNode = [nodeDic objectForKey:toId];
    if (!sNode || !eNode) {//起点或终点不在路径中，路径规划失败
        return nil;
    }
    AStarNode* start = [AStarNode instanceWithNodeId:sNode.nodeId preNode:nil gn:0 hn:0];
    [open addObject:start];
    
    BOOL success = NO;
    //A* 算法，操作open表
    while ([open count] > 0) {
        //从open表中获取fn最小的节点：可以使用遍历或者排序，鉴于排序代价较大，此次使用比较遍历
        AStarNode* minNode = [open firstObject];
        for (AStarNode* node in open) {
            if (node.fn < minNode.fn) {
                minNode = node;
            }
        }
        NSString* minNodeId = minNode.nodeId;
        //判断节点是否是终点，若是，A*算法进入递归算路阶段。
        if ([minNodeId isEqualToString:toId]) {
            [close addObject:minNode];
            success = YES;
            break;
        }
        
        //处理最近节点的相邻节点
        ALENavigationNode* minNaviNode = [nodeDic objectForKey:minNodeId];
        for (NSString* subId in minNaviNode.nextId) {
            //如果相邻节点已经在close表中，则直接忽略
            BOOL existInClose = NO;
            for (AStarNode* closeNode in close) {
                if ([closeNode.nodeId isEqualToString:subId]) {
                    existInClose = YES;
                    break;
                }
            }
            if (existInClose) {
                continue;
            }
            
            //计算从当前最小节点到达当前相邻节点的gn和当前相邻节点到达终点的fn值
            ALENavigationNode* subNaviNode = [nodeDic objectForKey:subId];
            double gn = minNode.gn + [ALEAStar gnCostFromNaviNode:minNaviNode toNaviNode:subNaviNode];
            double hn = [ALEAStar hnCostFromNaviNode:subNaviNode toNaviNode:eNode];
            
            //判断相邻节点是否在open表中
            AStarNode* nodeInOpen;
            for (AStarNode* openNode in open) {
                if ([openNode.nodeId isEqualToString:subId]) {
                    nodeInOpen = openNode;
                    break;
                }
            }
            //如果相邻节点已经在open表中，则需要判断是否需要更新其gn和hn值
            if (nodeInOpen) {
                if (nodeInOpen.fn > (gn + hn)) {
                    nodeInOpen.gn =  gn;
                    nodeInOpen.hn = hn;
                    nodeInOpen.preNode = minNode;
                }
            }else{//不在open表中，直接将该点放入open表
                AStarNode* subStarNode = [AStarNode instanceWithNodeId:subNaviNode.nodeId preNode:minNode gn:gn hn:hn];
                [open addObject:subStarNode];
            }
        }
        //本次对open表操作结束。将当前操作节点放入close表中
        [open removeObject:minNode];
        [close addObject:minNode];
        
    }
    if (!success) {//当open表遍历结束，也没有找到终点。算路失败
        return nil;
    }
    
    AStarNode* recNode = [close lastObject];
    //A*递归算路阶段
    NSMutableArray* retArray = [[NSMutableArray alloc] init];
    while (recNode.preNode) {
        NSString* nodeId = recNode.nodeId;
        ALENavigationNode* node = [nodeDic objectForKey:nodeId];
        if (node) {
            [retArray addObject:node];
        }
        recNode = recNode.preNode;
    }
    [retArray addObject:sNode];//加入起点
    //逆序排列
    NSMutableArray* mutRet = [[[retArray reverseObjectEnumerator] allObjects] mutableCopy];
    return mutRet;
}

#pragma mark - 启发函数

/**
 * 导航节点间的实际代价
 * 一张地图上，基准相同，实际代价目前使用两点间的距离（欧几里得距离）作为代价。
 **/
+(CGFloat)gnCostFromNaviNode:(ALENavigationNode*)startNaviNode toNaviNode:(ALENavigationNode*)toNaviNode{
    double ms_distance = [MathUtil distanceFromStartX:startNaviNode.x startY:startNaviNode.y endX:toNaviNode.x endY:toNaviNode.y];
    return ms_distance;
}
/**
 * 导航节点间的估算代价
 * 估算代价使用（曼哈顿距离： H(n) = D * (abs ( n.x – goal.x ) + abs ( n.y – goal.y ) )）
 **/
+(CGFloat)hnCostFromNaviNode:(ALENavigationNode*)startNaviNode toNaviNode:(ALENavigationNode*)toNaviNode{
    double ms_cost = fabs(toNaviNode.x - startNaviNode.x) + fabs(toNaviNode.y - startNaviNode.y);
    if (startNaviNode.x == toNaviNode.x || startNaviNode.y == toNaviNode.y) {
        ms_cost *= 1;
    }else{//如果有拐角的话，添加权值 ： 曼哈顿距离，D == 1.5
        ms_cost *= 1.5;
        //        ms_cost = MAX(fabs(toNaviNode.x - startNaviNode.x), fabs(toNaviNode.y - startNaviNode.y));//对角线距离 * D
    }
    return ms_cost;
}
@end
```

