---
layout:     post
title:      Mybatis常用的查询写法
date:       2019-07-03 17:50:12
author:     liangtong
catalog: true
categories: Java
tags: mybatis

---



记录下Mybatis中常用的查询xml文件写法。通常情况下，xml文件中的查询方法以select 、 count 开头



#### XML格式Mapper文件

格式如下，需要注意的是**mapper**接口文件的路径。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="eep.sippr.iiot.kanban.dao.ext.KanbanBoxExtMapper">
</mapper>
```



#### 数量统计（condition条件）

需要注意condition的配置及使用

+ condition 文件定义

  ```Java
  public class PlanEquipmentCondition {
    //此处省略属性及方法
  } 
  ```

+ mapper Java接口

  ```Java
  //查询设备计划数量
  Long countSumEquipmentPlanCount(@Param("condition") PlanEquipmentCountCondition condition);
  ```

+ mapper xml 文件写法

  ```xml
  <!--查询设备计划数量-->
      <select id="countSumEquipmentPlanCount"
              parameterType="eep.sippr.iiot.plan.condition.PlanEquipmentCountCondition"
              resultType="java.lang.Long">
          select sum(plan_equipment.plan_count) from plan_equipment
            left join plan_user
          on plan_equipment.user_plan_id = plan_user.user_plan_id
          <where>
              and plan_equipment.is_delete = 0
              and plan_equipment.is_validate = 1
              and plan_user.is_delete = 0
              and plan_user.is_validate = 1
              and (plan_equipment.process_unit_id is null or plan_equipment.is_last_process_unit = 1)
              <if test="condition.equipmentId != null">
                  and plan_equipment.equipment_id = #{condition.equipmentId}
              </if>
  
              <if test="condition.filterDateList != null">
                  and DATE_FORMAT(plan_user.date,'%Y-%m-%d') in
                  <foreach collection="condition.filterDateList"
                           item="item"
                           index="index"
                           open="(" close=")"
                           separator=",">
                      DATE_FORMAT(#{item},'%Y-%m-%d')
                  </foreach>
              </if>
          </where>
      </select>
  ```

#### 数量统计（简单条件）

需要注意xml方法中的参数的写法

+ mapper Java接口

  ```Java
      /***
       * 获取一级工单任务任务完成量
       ****/
      Long countPlanOrderReport1stProduceCount(@Param("planCode") String planCode);
      /***
       * 获取二级工单任务某一天的计划产量
       ****/
      Long countPlanOrder2stPlanCountByPlanLineIdAndDate(@Param("planLineId") String planLineId,
                                                         @Param("date") Date date);
  ```

  

+ mapper xml 文件写法

  ```xml
      <!--获取一级工单上报任务 - 完成产量 -->
      <select id="countPlanOrderReport1stProduceCount"
              parameterType="java.lang.String"
              resultType="java.lang.Long">
          select
          sum(plan_line.produce_count)
          from plan_line
          <where>
              and plan_line.is_delete = 0
              and plan_line.is_validate = 1
              <!--如果包含产线信息，则直接过滤产线相关-->
              <if test="planCode != null and planCode != ''">
                  and plan_line.plan_code = #{planCode}
              </if>
          </where>
      </select>
  
      <!--获取二级工单任务某一天的计划产量 -->
      <select id="countPlanOrder2stPlanCountByPlanLineIdAndDate"
              resultType="java.lang.Long">
          select
          sum(plan_user.plan_count)
          from plan_user
          left join plan_line
          on plan_line.plan_id = plan_user.plan_id
          <where>
              and plan_user.is_delete = 0
              and plan_user.is_validate = 1
              and plan_line.is_delete = 0
              and plan_line.is_validate = 1
              and (plan_user.process_unit_id is null or plan_user.is_last_process_unit = 1)
              <if test="planLineId != null and planLineId != ''">
                  and plan_line.plan_id = #{planLineId}
              </if>
              <if test="date != null">
                  <![CDATA[ and DATE_FORMAT(plan_user.date,'%Y-%m-%d') = DATE_FORMAT(#{date},'%Y-%m-%d')]]>
              </if>
          </where>
      </select>
  ```

#### 数据查询

需要注意 condition 及 返回值的写法

+ condition

  ```Java
  public class AbnormalReportCondition {
      private String factoryId;
      private String workshopId;
      private String lineId;
      private String equipmentId;
      private List<Date> filterDateList;
      // 具体代码隐藏
  }
  ```

+ mapper Java 接口

  ```Java
      /***
       * 查询产线级异常统计列表
       * 结果按照产线编码,异常状态编码排序
       **/
      List<AbnormalReportLineExt> selectAbnormalReportLineExtListByPage(@Param("condition") AbnormalReportCondition condition, RowBounds rowBounds);
  ```

+ mapper xml 实现

  ```xml
      <!--查询产线级异常统计列表-->
      <select id="selectAbnormalReportLineExtListByPage"
              parameterType="eep.sippr.iiot.andon.condition.AbnormalReportCondition"
              resultType="eep.sippr.iiot.andon.entity.ext.AbnormalReportLineExt">
  
          select
          line_id as lineId,
          exception_status as status,
          count(*) as count
  
          from exception_trigger
          <where>
              and is_validate = 1
              and is_delete = 0
              <if test="condition.factoryId != null">
                  and factory_id = #{condition.factoryId}
              </if>
              <if test="condition.workshopId != null">
                  and workshop_id = #{condition.workshopId}
              </if>
              <if test="condition.lineId != null">
                  and line_id = #{condition.lineId}
              </if>
              <if test="condition.equipmentId != null">
                  and equipment_id = #{condition.equipmentId}
              </if>
              <if test="condition.filterDateList != null">
                  and DATE_FORMAT(occur_date,'%Y-%m-%d') in
                  <foreach collection="condition.filterDateList"
                           item="item"
                           index="index"
                           open="(" close=")"
                           separator=",">
                      DATE_FORMAT(#{item},'%Y-%m-%d')
                  </foreach>
              </if>
          </where>
          group by line_id , exception_status
          order by line_id , exception_status
      </select>
  ```

#### 常用语法

+  choose

  ```xml
              <!--所在位置过滤-->
              <choose>
                  <!--CASE 1：如果存在设备编码，则无需过滤其他条件了-->
                  <when test="condition.equipmentId != null">
                      and A.equipment_id = #{condition.equipmentId}
                  </when>
                  <!--CASE 2：不存在设备编码，判断是否传递了产线编码-->
                  <when test="condition.lineId != null">
                      and A.line_id = #{condition.lineId}
                  </when>
                  <!--CASE 3：不存在产线编码，判断是否传递了车间编码-->
                  <when test="condition.workshopId != null">
                      and A.workshop_id = #{condition.workshopId}
                  </when>
                  <!--CASE 4：不存在车间编码，判断是否传递了工厂编码-->
                  <when test="condition.factoryId != null">
                      and A.factory_id = #{condition.factoryId}
                  </when>
                  <!--otherwise-->
                  <otherwise>
                    
                  </otherwise>
              </choose>
  ```

+ if 常量

  ```xml
              <if test="condition.filterBoxMac != false">
                  and box.screen_mac is not null
              </if>
  ```

+ case when

  ```xml
          select
          case when recovery_date is null then (select UNIX_TIMESTAMP(now()) - UNIX_TIMESTAMP(occur_date))
          else (select UNIX_TIMESTAMP(recovery_date) - UNIX_TIMESTAMP(occur_date))
          end
  ```

  

  

