# 20240912

记一次删除camunda历史变量表**act_hi_varinst**的操作。由于act_hi_varinst表的增长数据很快经过一年使用已经超过100GB，因为属于历史数据表所以对数据进行删除和备份操作，由于使用存储过程按天删除**act_hi_varinst**表，保留最近半年的数据。

1. 为act_hi_varinst表时间字段**CREATE_TIME_**添加索引，提高删除效率。
2. 使用存储过程按天删除数据，避免长时间阻塞事务，存储过程如下：

```mysql
-- 使用存储过程按照日期按天删除数据
DROP PROCEDURE IF EXISTS `p_webpub_dba_workflow_act_hi_varinst_delete`;
DELIMITER ;;
CREATE PROCEDURE `p_webpub_dba_workflow_act_hi_varinst_delete`()
BEGIN
    DECLARE _begin_time datetime DEFAULT '2023-09-11 01:00:00';
    DECLARE _end_time datetime DEFAULT '2024-03-11 01:00:00';
    loop_stop :
    LOOP
        IF _begin_time > _end_time THEN
            LEAVE loop_stop;
        END IF;
        delete from act_hi_varinst a where  a.CREATE_TIME_ < _begin_time;
        DO SLEEP(2);
        SET _begin_time = DATE_ADD(_begin_time, INTERVAL 1 DAY);
    END LOOP;
END
;;
DELIMITER ;;
```

3. 执行存储过程

```mysql
-- 执行存储过程
call p_webpub_dba_workflow_act_hi_varinst_delete();
```

