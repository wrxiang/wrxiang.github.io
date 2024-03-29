---
title: 用数据泵远程导出Oracle数据库
date: 2020-03-28 15:49:18
categories: Oracle
tags: [oracle, expdp]
---
## 应用场景
项目上做运维时，经常不知道客户数据库服务器远程访问的用户名和密码，只知道Oracle数据库的链接信息。Oracle11g以后数据库备份如果使用exp命令进行，经常有些表导出不成功或者出错，用expdp命令导出的话，备份文件又在服务器上，取不出来。这种情况下如果使用expdp命令导出，又想要备份文件保存在本地，则可以使用此方法。
前提：本地必须要有Oracle数据库，因为导出操作在本地数据库进行
操作目的：导出 10.96.37.30/orcl，用户为platform的数据库

<!--more-->

## 操作方法
在本地Oracle执行以下SQL命令：
```sql
--创建表空间
CREATE TABLESPACE BS_CKB DATAFILE 'S:/ORADATA/BSPHIS/BS_CKB.DBF' SIZE 5000M AUTOEXTEND ON NEXT 500M EXTENT MANAGEMENT LOCAL UNIFORM SIZE 256K;

--创建用户并赋予权限
CREATE USER BSCKB IDENTIFIED BY "bsoft" DEFAULT TABLESPACE BS_CKB  TEMPORARY TABLESPACE TEMP PROFILE DEFAULT;
GRANT CONNECT TO BSCKB;
GRANT DBA TO BSCKB;
GRANT IMP_FULL_DATABASE TO BSCKB;
GRANT RESOURCE TO BSCKB;
GRANT CREATE ANY SYNONYM TO BSCKB;
GRANT UNLIMITED TABLESPACE TO BSCKB;

--创建备份目录并授权
CREATE DIRECTORY DIR_BACKUP  AS 'S:\ORABACKUP\ORABACKUP_DMP';
GRANT READ,WRITE ON DIRECTORY DIR_BACKUP TO BSCKB;

--创建DB_LINK
CREATE PUBLIC DATABASE LINK LINK_BSCKB_PLATFORM
  CONNECT TO platform IDENTIFIED BY 密码
  USING '10.96.37.30/orcl';
```

PLSQL登录源数据库，执行以下命令赋予导出权限：
```sql
GRANT EXP_FULL_DATABASE TO platform;
```
在本地电脑的cmd中执行以下命令导出dmp文件
```bash
expdp BSCKB/bsoft@orcl directory=DIR_BACKUP dumpfile=BSCKB_PLATFORM.dump logfile=BSCKB_PLATFORM.log schemas=platform network_link=LINK_BSCKB_PLATFORM
```

踩坑记录：在执行导出命令时，提示 `ORA-39149: 无法将授权用户链接到非授权用户`，是由于没有将源数据库用户赋予 `EXP_FULL_DATABASE` 权限，赋予权限后还是报错提示没有权限
解决方法：创建db_link时直接使用dba用户创建，导出命令中指定需要导出的用户即可



