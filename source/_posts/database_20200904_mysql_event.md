---
layout:     post
title:      MYSQL 定时任务
date:       2020-09-04 19:25:13
author:     liangtong
catalog: true
categories: 数据库
tags: mysql

---

 需要简单3步，此处记录下：

+ 开启数据库任务调度

+ 创建存储过程

+ 开启定时器event



```mysql
-- 选择数据库
use db_xx_xx;

-- 开启事件
SET GLOBAL event_scheduler = ON;

-- 存储过程
DELIMITER //  
CREATE PROCEDURE p_planCron()
BEGIN
	-- 天数差值
	set @today = now();   
	set @start_day = date_add(@today , interval -31 day);  
	set @end_day = date_add(@today , interval -1 day);  
	set @dlta = datediff(@today,@end_day );

	-- 处理逻辑略

	-- 安全模式
	SET SQL_SAFE_UPDATES = 1;
END//
delimiter ;



-- EVENT
CREATE EVENT IF NOT EXISTS event_p_cron
	ON SCHEDULE EVERY 1 DAY STARTS DATE_ADD(DATE_ADD(CURDATE(), INTERVAL 1 DAY), INTERVAL 1 MINUTE)
	ON COMPLETION PRESERVE
	COMMENT '每天0点1分，更新演示数据'
	DO CALL p_planCron();
```

