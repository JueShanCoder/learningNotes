
# 性能测试报告

## 1. 测试目的

- 对比 Clickhouse、MySQL、ADB 之间处理同一复杂关联 SQL 的效率问题？
- 验证 Clickhouse Cluster 模式是否可以较 Standalone 模式大幅增快查询速率？（Not Yet ）



## 2. 测试方案

### 2.1 数据准备

- 采用 CRM 系统的 UAT 环境数据进行数据核验比对
- 以下是本次测试所到的数据表及表的数据量：

| TableName       | TableDataVolume | Comments     |
| --------------- | :-------------- | :----------- |
| t_clue          | 46368           | 线索主表     |
| t_clue_detail   | 328             | 线索详情表   |
| t_clue_log      | 482116          | 线索日志表   |
| t_clue_source   | 58848           | 线索来源表   |
| t_customer      | 16404           | 客户主表     |
| t_department    | 48              | 部门主表     |
| t_employee      | 59              | 员工主表     |
| t_intent_course | 32              | 意向课程表   |
| t_source        | 10              | 一级来源渠道 |
| t_source_detail | 96              | 二级来源渠道 |

## 3. 服务器配置

> Tips： MySQL 和 ADB 的测试环境为 CRM UAT环境

| MySQL    | ADB      | Clickhouse（Standalone） | Clickhouse（Cluster） |
| -------- | -------- | ------------------------ | --------------------- |
| 4 核 8 G | 4 核 16G | 4 核 8 G                 | Not Yet               |



## 4. 测试脚本

- **本次测试采用的是目前 CRM 业务上耗时较为明显的两条 SQL**

- > **SQL_1**

- > **SQL_2**



## 5. 测试结果

### 5.1 SQL_1 测试结果（S）

| ADB     | MySQL     | Clickhouse（Standalone） | Clickhouse（Cluster） |
| ------- | --------- | ------------------------ | --------------------- |
| 5.558 S | 203.158 S | 1.159 S                  | Not Yet               |



### 5.2 SQL_2 测试结果（s）

| ADB     | MySQL    | Clickhouse（Standalone） | Clickhouse（Cluster） |
| ------- | -------- | ------------------------ | --------------------- |
| 4.076 S | 21.247 S | 0.792 S                  | Not Yet               |



## 6. 测试结论

MySQL耗时最长，内存占用最大，ADB 居中，Clickhouse （Standalone）最优。



## 7. 参考文档

[Clickhouse官方文档](https://clickhouse.tech/docs/zh/)

****

[ADB 官方文档](https://www.aliyun.com/product/ApsaraDB/ads)

****

[^SQL_1]:`SELECT dt0.sourceId AS sourceId, dt0.sourceName AS sourceName, IFNULL(COUNT(DISTINCT(CASE WHEN dt0.create_type NOT IN (13, 14) AND (toDateTime(dt0.created) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') THEN dt0.id ELSE NULL END)), 0) AS numOfNewClues, IFNULL(COUNT(DISTINCT(CASE WHEN dt0.create_type IN (13, 14) AND (toDateTime(dt0.created) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') THEN dt0.id ELSE NULL END)), 0) AS seaPickUp, IFNULL(COUNT(DISTINCT(CASE WHEN dt0.follow_status IN (1, 2, 3, 8) AND (toDateTime(dt0.created) <= '2021-03-31 00:00:00') THEN dt0.id ELSE NULL END)), 0) AS holdClueNum, IFNULL(COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.signed_up_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.id ELSE NULL END)), 0) AS numOfApplicants, IFNULL(COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.success_closed_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.id ELSE NULL END)), 0) AS successCloseOrder, IFNULL(SUM(CASE WHEN (dt0.amount > 0 OR (dt0.amount < 0 AND dt0.category IN (6, 8))) AND (toDateTime(dt0.success_closed_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.amount ELSE NULL END), 0) AS receivedAmount, IFNULL(COUNT(DISTINCT(CASE WHEN dt0.follow_status = 7 AND (toDateTime(dt0.refund_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.id ELSE NULL END)), 0) AS numOfRefund, IFNULL(SUM(CASE WHEN dt0.follow_status = 7 AND (toDateTime(dt0.refund_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN (dt0.refund_amount) ELSE NULL END), 0) + IFNULL(SUM(CASE WHEN dt0.follow_status = 7 AND (toDateTime(dt0.refund_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN (dt0.deposit_refund_amount) ELSE NULL END), 0) AS refundAmount, ROUND((IFNULL( ( COUNT(DISTINCT(CASE WHEN toDateTime(dt0.signed_up_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00' AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.id ELSE NULL END )) / COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.created) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') THEN dt0.id ELSE NULL END)) ), 0) * 100), 2) AS reConversionRate, ROUND((IFNULL( ( COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.signed_up_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') AND DATEDIFF('day',toDateTime(dt0.signed_up_at), toDateTime(dt0.belonged)) <= 7 AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.id ELSE NULL END )) / COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.created) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') THEN dt0.id ELSE NULL END)) ), 0) * 100), 2) AS reSevenConversionRate, ROUND((IFNULL( ( COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.signed_up_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') AND DATEDIFF('day',toDateTime(dt0.signed_up_at), toDateTime(dt0.belonged)) <= 30 AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.id ELSE NULL END )) / COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.created) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') THEN dt0.id ELSE NULL END)) ), 0) * 100), 2) AS reThirtyConversionRate, ROUND((IFNULL( ( COUNT(DISTINCT(CASE WHEN toDateTime(dt0.success_closed_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00' AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.id ELSE NULL END )) / COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.created) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') THEN dt0.id ELSE NULL END)) ), 0) * 100), 2) AS sCConversionRate, ROUND((IFNULL( ( COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.success_closed_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') AND DATEDIFF('day',toDateTime(dt0.success_closed_at), toDateTime(dt0.belonged)) <= 7 AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.id ELSE NULL END )) / COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.created) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') THEN dt0.id ELSE NULL END)) ), 0) * 100), 2) AS sCSevenConversionRate, ROUND((IFNULL( ( COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.success_closed_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') AND DATEDIFF('day',toDateTime(dt0.success_closed_at), toDateTime(dt0.belonged)) <= 30 AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.id ELSE NULL END )) / COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.created) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') THEN dt0.id ELSE NULL END)) ), 0) * 100), 2) AS sCThirtyConversionRate, ROUND((IFNULL( ( COUNT(DISTINCT(CASE WHEN dt0.follow_status = 7 AND ((toDateTime(dt0.signed_up_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') OR ((toDateTime(dt0.paid_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') AND dt0.op_type = 2 AND dt0.follow_status_log = 3 AND toInt8(dt0.payable_amount) > dt0.paid_amount)) AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.id ELSE NULL END )) / COUNT(DISTINCT(CASE WHEN ((toDateTime(dt0.signed_up_at) <= '2021-03-31 00:00:00') OR ((toDateTime(dt0.paid_at) <= '2021-03-31 00:00:00') AND dt0.op_type = 2 AND dt0.follow_status_log = 3 AND toInt8(dt0.payable_amount) > dt0.paid_amount)) AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.id ELSE NULL END )) ), 0) * 100), 2) AS lossOfOrderConversionRate, 0 AS effectiveCluesRate, 0 AS newClueRate FROM ( SELECT DISTINCT c1.id AS id, l1.id AS log_id, e1.id AS consultantId, e1.real_name AS consultantName, t1.id AS departmentId, t1.name AS departmentName, l1.op_type AS op_type, l1.paid_at/1000 AS paid_at, l1.payable_amount/100 AS payable_amount, l1.paid_amount/100 AS paid_amount, c1.signed_up_at/1000 AS signed_up_at, c1.create_type AS create_type, c1.created/1000 AS created, c1.follow_status AS follow_status, l1.follow_status AS follow_status_log, l1.amount/100 AS amount, l1.refund_at/1000 AS refund_at, s2.id AS sourceId, s2.name AS sourceName, l1.refund_amount/100 AS refund_amount, s3.id AS sourceDetailId, s3.name AS sourceDetailName, d1.success_closed_at/1000 AS success_closed_at, c1.belonged/1000 AS belonged, i1.type AS type1, i2.type AS type2, l1.deposit_refund_amount/100 AS deposit_refund_amount, l1.category AS category FROM t_clue c1 LEFT JOIN t_clue_log l1 ON c1.id = l1.clue_id LEFT JOIN t_clue_detail d1 ON c1.id = d1.clue_id LEFT JOIN t_intent_course i1 ON c1.enrolment_course_id = i1.course_id LEFT JOIN t_intent_course i2 ON d1.order_course_id = i2.course_id LEFT JOIN t_clue_source s1 ON c1.clue_source_id = s1.id LEFT JOIN t_source s2 ON s1.source_id = s2.id LEFT JOIN t_employee e1 ON c1.belonger_id = e1.id LEFT JOIN t_department t1 ON e1.department_id = t1.id LEFT JOIN t_source_detail s3 ON s1.source_detail_id = s3.id WHERE 1=1 AND c1.deleted = 0 AND c1.create_status IN ( 1,3,4 ) AND t1.id IN ( 3 , 4 , 5 ) ) dt0 GROUP BY dt0.sourceId,dt0.sourceName ORDER BY dt0.sourceId`

****

[^SQL_2]:`SELECT IFNULL(COUNT(DISTINCT(CASE WHEN dt0.create_type NOT IN (13, 14) AND (toDateTime(dt0.created) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') THEN dt0.id ELSE NULL END)), 0) AS numOfNewClues, IFNULL(COUNT(DISTINCT(CASE WHEN dt0.create_type IN (13, 14) AND (toDateTime(dt0.created) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') THEN dt0.id ELSE NULL END)), 0) AS seaPickUp, IFNULL(COUNT(DISTINCT(CASE WHEN dt0.follow_status IN (1, 2, 3, 8) AND (toDateTime(dt0.created) <= '2021-03-31 00:00:00') THEN dt0.id ELSE NULL END)), 0) AS holdClueNum, IFNULL(COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.signed_up_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.id ELSE NULL END)), 0) AS numOfApplicants, IFNULL(COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.success_closed_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.id ELSE NULL END)), 0) AS successCloseOrder, IFNULL(SUM(CASE WHEN (dt0.amount > 0 OR (dt0.amount < 0 AND dt0.category IN (6, 8))) AND (toDateTime(dt0.success_closed_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.amount ELSE NULL END), 0) AS receivedAmount, IFNULL(COUNT(DISTINCT(CASE WHEN dt0.follow_status = 7 AND (toDateTime(dt0.refund_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.id ELSE NULL END)), 0) AS numOfRefund, IFNULL(SUM(CASE WHEN dt0.follow_status = 7 AND (toDateTime(dt0.refund_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN (dt0.refund_amount) ELSE NULL END), 0) + IFNULL(SUM(CASE WHEN dt0.follow_status = 7 AND (toDateTime(dt0.refund_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN (dt0.deposit_refund_amount) ELSE NULL END), 0) AS refundAmount, ROUND((IFNULL( ( COUNT(DISTINCT(CASE WHEN toDateTime(dt0.signed_up_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00' AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.id ELSE NULL END )) / COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.created) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') THEN dt0.id ELSE NULL END)) ), 0) * 100), 2) AS reConversionRate, ROUND((IFNULL( ( COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.signed_up_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') AND DATEDIFF('day',toDateTime(dt0.signed_up_at), toDateTime(dt0.belonged)) <= 7 AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.id ELSE NULL END )) / COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.created) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') THEN dt0.id ELSE NULL END)) ), 0) * 100), 2) AS reSevenConversionRate, ROUND((IFNULL( ( COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.signed_up_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') AND DATEDIFF('day',toDateTime(dt0.signed_up_at), toDateTime(dt0.belonged)) <= 30 AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.id ELSE NULL END )) / COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.created) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') THEN dt0.id ELSE NULL END)) ), 0) * 100), 2) AS reThirtyConversionRate, ROUND((IFNULL( ( COUNT(DISTINCT(CASE WHEN toDateTime(dt0.success_closed_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00' AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.id ELSE NULL END )) / COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.created) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') THEN dt0.id ELSE NULL END)) ), 0) * 100), 2) AS sCConversionRate, ROUND((IFNULL( ( COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.success_closed_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') AND DATEDIFF('day',toDateTime(dt0.success_closed_at), toDateTime(dt0.belonged)) <= 7 AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.id ELSE NULL END )) / COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.created) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') THEN dt0.id ELSE NULL END)) ), 0) * 100), 2) AS sCSevenConversionRate, ROUND((IFNULL( ( COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.success_closed_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') AND DATEDIFF('day',toDateTime(dt0.success_closed_at), toDateTime(dt0.belonged)) <= 30 AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.id ELSE NULL END )) / COUNT(DISTINCT(CASE WHEN (toDateTime(dt0.created) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') THEN dt0.id ELSE NULL END)) ), 0) * 100), 2) AS sCThirtyConversionRate, ROUND((IFNULL( ( COUNT(DISTINCT(CASE WHEN dt0.follow_status = 7 AND ((toDateTime(dt0.signed_up_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') OR ((toDateTime(dt0.paid_at) BETWEEN '2021-03-01 00:00:00' AND '2021-03-31 00:00:00') AND dt0.op_type = 2 AND dt0.follow_status_log = 3 AND toInt8(dt0.payable_amount) > dt0.paid_amount)) AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.id ELSE NULL END )) / COUNT(DISTINCT(CASE WHEN ((toDateTime(dt0.signed_up_at) <= '2021-03-31 00:00:00') OR ((toDateTime(dt0.paid_at) <= '2021-03-31 00:00:00') AND dt0.op_type = 2 AND dt0.follow_status_log = 3 AND toInt8(dt0.payable_amount) > dt0.paid_amount)) AND ( dt0.type1 IN ( 1 , 2 , 3 , 4 ) OR dt0.type2 IN ( 1 , 2 , 3 , 4 ) ) THEN dt0.id ELSE NULL END )) ), 0) * 100), 2) AS lossOfOrderConversionRate, 0 AS effectiveCluesRate, 0 AS newClueRate FROM ( SELECT DISTINCT c1.id AS id, l1.id AS log_id, e1.id AS consultantId, e1.real_name AS consultantName, t1.id AS departmentId, t1.name AS departmentName, l1.op_type AS op_type, l1.paid_at/1000 AS paid_at, l1.payable_amount/100 AS payable_amount, l1.paid_amount/100 AS paid_amount, c1.signed_up_at/1000 AS signed_up_at, c1.create_type AS create_type, c1.created/1000 AS created, c1.follow_status AS follow_status, l1.follow_status AS follow_status_log, l1.amount/100 AS amount, l1.refund_at/1000 AS refund_at, l1.refund_amount/100 AS refund_amount, d1.success_closed_at/1000 AS success_closed_at, c1.belonged/1000 AS belonged, i1.type AS type1, i2.type AS type2, l1.deposit_refund_amount/100 AS deposit_refund_amount, l1.category AS category FROM t_clue c1 LEFT JOIN t_clue_log l1 ON c1.id = l1.clue_id LEFT JOIN t_clue_detail d1 ON c1.id = d1.clue_id LEFT JOIN t_intent_course i1 ON c1.enrolment_course_id = i1.course_id LEFT JOIN t_intent_course i2 ON d1.order_course_id = i2.course_id LEFT JOIN t_clue_source s1 ON c1.clue_source_id = s1.id LEFT JOIN t_employee e1 ON c1.belonger_id = e1.id LEFT JOIN t_department t1 ON e1.department_id = t1.id WHERE 1 = 1 AND c1.create_status IN ( 1,3,4 ) AND c1.deleted = 0 AND s1.source_id IN ( 1 , 2 , 3 , 4 , 5 , 6 ) AND s1.source_detail_id IN ( 101 , 102 , 103 , 104 , 105 , 106 , 201 , 202 , 203 , 204 , 205 , 206 , 207 , 301 , 302 , 303 , 304 , 305 , 306 , 307 , 308 , 401 , 402 , 403 , 404 , 405 , 406 , 407 , 408 , 409 , 410 , 411 , 412 , 413 , 414 , 415 , 501 , 502 , 503 , 504 , 505 , 506 , 507 , 508 , 509 , 601 , 602 , 603 , 604 , 605 , 606 , 607 , 608 , 609 , 610 , 611 , 612 ) AND t1.id IN ( 3 , 4 , 5 ) ) dt0 `

