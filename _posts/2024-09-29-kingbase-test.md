---
layout: post
title: "人大金仓数据库测试"
date: 2024-09-29
tags: [java,国产化,数据库]
---
# kingbase人大金仓测试

官方文档

https://help.kingbase.com.cn/v8/index.html

下载地址

https://www.kingbase.com.cn/xzzx/index.htm

## docker run



![image-20240801172600062](https://s3.cdn.q1sj.cn/2024/08/58a2afb7f1326aa76c7e14b37a6b0d4b.png)



```shell
sudo docker load -i kdb_x86_64_V008R006C008B0020.tar

sudo docker run -tid --privileged --restart=always \
-p 54321:54321 \
-e DB_USER=kingbase  \
-e DB_PASSWORD=123456 \
-e DB_MODE=mysql  \
--name kingbase  \
-v /root/kingbase/data:/home/kingbase/userdata/ \
kingbase_v008r006c008b0020_single_x86:v1 /usr/sbin/init
```
## sql

```sql

alter system set search_path = "$USER", PUBLIC,SYS_CATALOG;
select sys_reload_conf();

create user demo password '123456'
grant kingbase to demo

select get_license_info();
```

## ddl对比

### mysql

```sql
CREATE TABLE `toll_station_ramp`  (
    `id` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
    `scene` int(4) NOT NULL DEFAULT 0 COMMENT '场景：1收费站进口外、2收费站进口内、3收费站出口外、4收费站出口内',
    `ramp_no` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '' COMMENT '匝道编号',
    `ramp_name` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '' COMMENT '匝道名称',
    `ave_speed` decimal(10, 6) NOT NULL COMMENT '平均车速（单位：千米每小时）',
    `queue_length` int(8) NOT NULL DEFAULT 0 COMMENT '排队长度（单位：米）',
    `drive_time` int(8) NOT NULL DEFAULT 0 COMMENT '预计通过时长（单位：分钟）',
    `remark` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '' COMMENT '备注',
    `scene_name` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '' COMMENT '场景名称',
    `report_time` datetime(0) NOT NULL COMMENT '上报时间',
    `create_time` datetime(0) NOT NULL COMMENT '创建时间',
    `sampling` bit(1) NOT NULL DEFAULT b'0' COMMENT '是否采样',
    `push_province_success` bit(1) NOT NULL DEFAULT b'0' COMMENT '推送省级是否成功',
    `push_road_success` bit(1) NOT NULL DEFAULT b'0' COMMENT '推送路段是否成功',
    `extinfo` text CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL COMMENT '预留字段',
    PRIMARY KEY (`id`) USING BTREE,
    INDEX `idx_ramp_no`(`ramp_no`) USING BTREE,
    INDEX `idx_report_time`(`report_time`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci COMMENT = '匝道数据记录表' ROW_FORMAT = Dynamic;
```

### kingbase

删除 `ENGINE=InnoDB AUTO_INCREMENT=864 DEFAULT CHARSET=utf8mb4`

`varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL` 修改为 `varchar(64)`

数据类型 `bit(1)` 需要改为 `bool`

```sql
CREATE TABLE `toll_station_ramp`  (
    `id` varchar(64) NOT NULL,
    `scene` int(4) NOT NULL DEFAULT 0 COMMENT '场景：1收费站进口外、2收费站进口内、3收费站出口外、4收费站出口内',
    `ramp_no` varchar(64) NOT NULL DEFAULT '' COMMENT '匝道编号',
    `ramp_name` varchar(64) NOT NULL DEFAULT '' COMMENT '匝道名称',
    `ave_speed` decimal(10, 6) NOT NULL COMMENT '平均车速（单位：千米每小时）',
    `queue_length` int(8) NOT NULL DEFAULT 0 COMMENT '排队长度（单位：米）',
    `drive_time` int(8) NOT NULL DEFAULT 0 COMMENT '预计通过时长（单位：分钟）',
    `remark` varchar(255) NOT NULL DEFAULT '' COMMENT '备注',
    `scene_name` varchar(64) NOT NULL DEFAULT '' COMMENT '场景名称',
    `report_time` datetime(0) NOT NULL COMMENT '上报时间',
    `create_time` datetime(0) NOT NULL COMMENT '创建时间',
    `sampling` bool NOT NULL DEFAULT b'0' COMMENT '是否采样',
    `push_province_success` bool NOT NULL DEFAULT b'0' COMMENT '推送省级是否成功',
    `push_road_success` bool NOT NULL DEFAULT b'0' COMMENT '推送路段是否成功',
    `extinfo` text NULL COMMENT '预留字段',
    PRIMARY KEY (`id`) USING BTREE,
    INDEX `idx_ramp_no`(`ramp_no`) USING BTREE,
    INDEX `idx_report_time`(`report_time`) USING BTREE
);
```



## pom.xml

```xml
<dependency>
    <groupId>cn.com.kingbase</groupId>
    <artifactId>kingbase8</artifactId>
    <version>8.6.0</version>
</dependency>
```



## appliction.yml

```yaml
spring:
  datasource:
    driver-class-name: com.kingbase8.Driver
    url: jdbc:kingbase8://192.168.247.130:54321/kingbase?currentSchema=public,sys_catalog&useSSL=false
    username: kingbase
    password: 123456
```



## 使用navicat连接数据库

![image-20240801093343059](https://s3.cdn.q1sj.cn/2024/08/1d11ecd2189e7323cf1c1b6d7dff45cc.png)

![image-20240801093304669](https://s3.cdn.q1sj.cn/2024/08/bb9f1b9a783be1052bc97be5dbf15725.png)





## 海光服务器安装kingbase

### 麒麟系统

docker安装无法使用host网络模式  其他和linux x86相同





