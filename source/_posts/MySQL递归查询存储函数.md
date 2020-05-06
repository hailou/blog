title: MySQL递归查询存储函数
categories: 代码案例
tags:
  - Git
date: 2020-05-06 15:02:13
---
###### 向下查询（根据父级查询子级）
```sql
select * from tb_base_region WHERE FIND_IN_SET(region_code,getRegionCodeListForDown('01'));
```
###### 向上查询（根据子级查询父级）
```sql
select * from tb_base_region WHERE FIND_IN_SET(region_code,getRegionCodeListForUp('010101'));
```
<!-- more -->

###### 创建向下查询的存储函数，查询出符合要求的数据的region_code集合，以逗号拼接成字符串。
```sql
CREATE DEFINER=`skip-grants user`@`skip-grants host` FUNCTION `getRegionCodeListForDown`(upCode VARCHAR(10)) RETURNS varchar(4096) CHARSET utf8
BEGIN
DECLARE returnCodeList VARCHAR(4096);
DECLARE selectCode VARCHAR(4096);
SET returnCodeList = '';
SET selectCode = upCode;
WHILE selectCode IS NOT NULL DO
SET returnCodeList = CONCAT(returnCodeList,',',selectCode);
select GROUP_CONCAT(region_code) INTO selectCode FROM tb_base_region WHERE FIND_IN_SET(parent_code,selectCode) > 0;
END WHILE;
RETURN returnCodeList;
END;
```
###### 创建向上查询的存储函数，查询出符合要求的数据的region_code集合。
```sql
CREATE  DEFINER=`skip-grants user`@`skip-grants host` FUNCTION `getRegionCodeListForUp`(downCode VARCHAR(10)) RETURNS varchar(4096) CHARSET utf8
BEGIN
DECLARE returnCodeList VARCHAR(4096);
DECLARE selectCode VARCHAR(4096);
SET returnCodeList = '';
SET selectCode = downCode;
-- WHILE selectCode IS NOT NULL DO  -- 判断条件视情况而定
WHILE selectCode > 0 AND downId in (select downCode from tb_base_region) DO
SET returnCodeList = CONCAT(returnCodeList,',',selectCode);
SELECT parent_code into selectCode from tb_base_region WHERE region_code = selectCode;
END WHILE;
RETURN returnCodeList;
END;
```
