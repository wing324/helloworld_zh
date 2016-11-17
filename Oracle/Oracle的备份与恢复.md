## Oracle的备份与恢复

数据库中最重要最关键的就是备份与恢复啦，这节我们就来谈一谈Oralce数据库的备份与恢复。  



针对expdp,impdp的使用。  
参考链接：  
https://oracle-base.com/articles/10g/oracle-data-pump-10g  


创建示例数据
-----------
以scott用户为例  
```sql
[oracle@wing ~]$ sqlplus / as sysdba
SQL*Plus: Release 11.2.0.4.0 Production on Fri Feb 26 10:52:51 2016
Copyright (c) 1982, 2013, Oracle.  All rights reserved.
Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
SQL>
# 创建目录
SQL> create directory test_dir as '/u01/backup'; 
Directory created
# 给scott用户解锁并设置密码
SQL> ALTER USER scott IDENTIFIED BY scott ACCOUNT UNLOCK;
# 给scott用户赋权
SQL> grant read,write on directory test_dir to scott;
Grant succeeded.

# 以scott用户登录数据库中
[oracle@wing ~]$ sqlplus scott/scott@WINGDB
SQL*Plus: Release 11.2.0.4.0 Production on Fri Feb 26 11:09:15 2016
Copyright (c) 1982, 2013, Oracle.  All rights reserved.
Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
SQL> 

# 创建表并且插入数据
SQL> create table test_dump (id int);
Table created
SQL> commit;
Commit complete
SQL> desc test_dump;
Name Type    Nullable Default Comments 
---- ------- -------- ------- -------- 
ID   INTEGER Y                         
SQL> insert into test_dump values(1);
1 row inserted
SQL> insert into test_dump values(2);
1 row inserted
SQL> insert into test_dump values(3);
1 row inserted
SQL> insert into test_dump values(4);
1 row inserted
SQL> insert into test_dump values(5);
1 row inserted
SQL> insert into test_dump values(6);
1 row inserted
SQL> insert into test_dump values(7);
1 row inserted
SQL> insert into test_dump values(8);
1 row inserted
SQL> insert into test_dump values(9);
1 row inserted
SQL> insert into test_dump values(10);
1 row inserted
SQL> commit;
Commit complete
SQL> select * from test_dump;
                                     ID
---------------------------------------
                                      1
                                      2
                                      3
                                      4
                                      5
                                      6
                                      7
                                      8
                                      9
                                     10
10 rows selected
SQL> create table test (name varchar(16));
Table created
SQL> desc test;
Name Type         Nullable Default Comments 
---- ------------ -------- ------- -------- 
NAME VARCHAR2(16) Y                         
SQL> insert into test values('Oracle');
1 row inserted
SQL> insert into test values('MySQL');
1 row inserted
SQL> insert into test values('MS SQL');
1 row inserted
SQL> insert into test values('SQL server');
1 row inserted
SQL> insert into test values('PostgreSQL');
1 row inserted
SQL> commit;
Commit complete
SQL> select * from test;
NAME
----------------
Oracle
MySQL
MS SQL
SQL server
PostgreSQL

SQL> create table scott_table (id int,name varchar(16));
Table created
SQL> insert into scott_table values(1,'Oracle');
1 row inserted
SQL> insert into scott_table values(2,'MySQL');
1 row inserted
SQL> commit;
Commit complete

# 至此，示例表创建完毕，共有test,test_dump,scott_table三张表
```


expdp
-----
###### 表导出
以test_dump,scott_table表导出为例子
```
# tables指定需要备份的表
# directory存放dumpfile和logfile的地方
# dumpfile备份文件
# logfile备份日志文件
# 备份scott用户的test_dump,scott_table表
[oracle@wing ~]$ expdp scott/scott@WINGDB tables=test_dump,scott_table directory=TEST_DIR dumpfile=scott_dump.dmp logfile=scott_exp.log
Export: Release 11.2.0.4.0 - Production on Fri Feb 26 11:45:04 2016
Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.
Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
Starting "SCOTT"."SYS_EXPORT_TABLE_01":  scott/********@WINGDB tables=test_dump,scott_table directory=TEST_DIR dumpfile=scott_dump.dmp logfile=scott_exp.log 
Estimate in progress using BLOCKS method...
Processing object type TABLE_EXPORT/TABLE/TABLE_DATA
Total estimation using BLOCKS method: 128 KB
Processing object type TABLE_EXPORT/TABLE/TABLE
. . exported "SCOTT"."SCOTT_TABLE"                       5.445 KB       2 rows
. . exported "SCOTT"."TEST_DUMP"                         5.085 KB      10 rows
Master table "SCOTT"."SYS_EXPORT_TABLE_01" successfully loaded/unloaded
******************************************************************************
Dump file set for SCOTT.SYS_EXPORT_TABLE_01 is:
  /u01/backup/scott_dump.dmp
Job "SCOTT"."SYS_EXPORT_TABLE_01" successfully completed at Fri Feb 26 11:45:21 2016 elapsed 0 00:00:14

# 进入directory目录下查看存在dumpfile和logfile
[root@wing backup]# ll
total 112
-rw-r-----. 1 oracle oinstall 110592 Feb 26 11:45 scott_dump.dmp
-rw-r--r--. 1 oracle oinstall   1164 Feb 26 11:45 scott_exp.log
```

###### schema导出
以scottschema为例  
```
[oracle@wing ~]$ expdp scott/scott@WINGDB schemas=SCOTT directory=TEST_DIR dumpfile=SCOTT.dmp logfile=scott_exp.log

Export: Release 11.2.0.4.0 - Production on Fri Feb 26 13:30:13 2016

Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
Starting "SCOTT"."SYS_EXPORT_SCHEMA_01":  scott/********@WINGDB schemas=SCOTT directory=TEST_DIR dumpfile=SCOTT.dmp logfile=scott_exp.log 
Estimate in progress using BLOCKS method...
Processing object type SCHEMA_EXPORT/TABLE/TABLE_DATA
Total estimation using BLOCKS method: 384 KB
Processing object type SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
Processing object type SCHEMA_EXPORT/TABLE/TABLE
Processing object type SCHEMA_EXPORT/TABLE/COMMENT
Processing object type SCHEMA_EXPORT/TABLE/INDEX/INDEX
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
Processing object type SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/INDEX_STATISTICS
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT
Processing object type SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
. . exported "SCOTT"."DEPT"                              5.929 KB       4 rows
. . exported "SCOTT"."EMP"                               8.562 KB      14 rows
. . exported "SCOTT"."SALGRADE"                          5.859 KB       5 rows
. . exported "SCOTT"."SCOTT_TABLE"                       5.468 KB       4 rows
. . exported "SCOTT"."TEST"                              5.062 KB       5 rows
. . exported "SCOTT"."TEST_DUMP"                         5.156 KB      20 rows
. . exported "SCOTT"."BONUS"                                 0 KB       0 rows
Master table "SCOTT"."SYS_EXPORT_SCHEMA_01" successfully loaded/unloaded
******************************************************************************
Dump file set for SCOTT.SYS_EXPORT_SCHEMA_01 is:
  /u01/backup/SCOTT.dmp
Job "SCOTT"."SYS_EXPORT_SCHEMA_01" successfully completed at Fri Feb 26 13:30:48 2016 elapsed 0 00:00:34

```

###### database导出
整个数据库的导出  
```
# full=Y代表全备整个数据库
[oracle@wing ~]$ expdp system/Wing_database@WINGDB full=Y directory=TEST_DIR dumpfile=oracle11g.dmp logfile=oracle11g_exp.log
Export: Release 11.2.0.4.0 - Production on Fri Feb 26 13:57:33 2016
Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.
Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
Starting "SYSTEM"."SYS_EXPORT_FULL_01":  system/********@WINGDB full=Y directory=TEST_DIR dumpfile=oracle11g.dmp logfile=oracle11g_exp.log 
Estimate in progress using BLOCKS method...
Processing object type DATABASE_EXPORT/SCHEMA/TABLE/TABLE_DATA
Total estimation using BLOCKS method: 384.6 MB
Processing object type DATABASE_EXPORT/TABLESPACE
Processing object type DATABASE_EXPORT/PROFILE
Processing object type DATABASE_EXPORT/SYS_USER/USER
Processing object type DATABASE_EXPORT/SCHEMA/USER
Processing object type DATABASE_EXPORT/ROLE
Processing object type DATABASE_EXPORT/GRANT/SYSTEM_GRANT/PROC_SYSTEM_GRANT
Processing object type DATABASE_EXPORT/SCHEMA/GRANT/SYSTEM_GRANT
Processing object type DATABASE_EXPORT/SCHEMA/ROLE_GRANT
Processing object type DATABASE_EXPORT/SCHEMA/DEFAULT_ROLE
Processing object type DATABASE_EXPORT/SCHEMA/TABLESPACE_QUOTA
Processing object type DATABASE_EXPORT/RESOURCE_COST
Processing object type DATABASE_EXPORT/TRUSTED_DB_LINK
Processing object type DATABASE_EXPORT/SCHEMA/SEQUENCE/SEQUENCE
Processing object type DATABASE_EXPORT/SCHEMA/SEQUENCE/GRANT/OWNER_GRANT/OBJECT_GRANT
Processing object type DATABASE_EXPORT/DIRECTORY/DIRECTORY
Processing object type DATABASE_EXPORT/DIRECTORY/GRANT/OWNER_GRANT/OBJECT_GRANT
Processing object type DATABASE_EXPORT/CONTEXT
Processing object type DATABASE_EXPORT/SCHEMA/PUBLIC_SYNONYM/SYNONYM
Processing object type DATABASE_EXPORT/SCHEMA/SYNONYM
Processing object type DATABASE_EXPORT/SCHEMA/TYPE/INC_TYPE
Processing object type DATABASE_EXPORT/SCHEMA/TYPE/TYPE_SPEC
Processing object type DATABASE_EXPORT/SCHEMA/TYPE/GRANT/OWNER_GRANT/OBJECT_GRANT
Processing object type DATABASE_EXPORT/SYSTEM_PROCOBJACT/PRE_SYSTEM_ACTIONS/PROCACT_SYSTEM
Processing object type DATABASE_EXPORT/SYSTEM_PROCOBJACT/PROCOBJ
Processing object type DATABASE_EXPORT/SYSTEM_PROCOBJACT/POST_SYSTEM_ACTIONS/PROCACT_SYSTEM
Processing object type DATABASE_EXPORT/SCHEMA/PROCACT_SCHEMA
Processing object type DATABASE_EXPORT/SCHEMA/XMLSCHEMA/XMLSCHEMA
Processing object type DATABASE_EXPORT/SCHEMA/TABLE/TABLE
Processing object type DATABASE_EXPORT/SCHEMA/TABLE/PRE_TABLE_ACTION
Processing object type DATABASE_EXPORT/SCHEMA/TABLE/GRANT/OWNER_GRANT/OBJECT_GRANT
Processing object type DATABASE_EXPORT/SCHEMA/TABLE/COMMENT
Processing object type DATABASE_EXPORT/SCHEMA/PACKAGE/PACKAGE_SPEC
Processing object type DATABASE_EXPORT/SCHEMA/PACKAGE/GRANT/OWNER_GRANT/OBJECT_GRANT
Processing object type DATABASE_EXPORT/SCHEMA/FUNCTION/FUNCTION
Processing object type DATABASE_EXPORT/SCHEMA/FUNCTION/GRANT/OWNER_GRANT/OBJECT_GRANT
Processing object type DATABASE_EXPORT/SCHEMA/PROCEDURE/PROCEDURE
Processing object type DATABASE_EXPORT/SCHEMA/PROCEDURE/GRANT/OWNER_GRANT/OBJECT_GRANT
Processing object type DATABASE_EXPORT/SCHEMA/PACKAGE/COMPILE_PACKAGE/PACKAGE_SPEC/ALTER_PACKAGE_SPEC
Processing object type DATABASE_EXPORT/SCHEMA/FUNCTION/ALTER_FUNCTION
Processing object type DATABASE_EXPORT/SCHEMA/PROCEDURE/ALTER_PROCEDURE
Processing object type DATABASE_EXPORT/SCHEMA/TABLE/INDEX/INDEX
Processing object type DATABASE_EXPORT/SCHEMA/TABLE/INDEX/FUNCTIONAL_INDEX/INDEX
Processing object type DATABASE_EXPORT/SCHEMA/TABLE/CONSTRAINT/CONSTRAINT
Processing object type DATABASE_EXPORT/SCHEMA/TABLE/INDEX/STATISTICS/INDEX_STATISTICS
Processing object type DATABASE_EXPORT/SCHEMA/TABLE/INDEX/STATISTICS/FUNCTIONAL_INDEX/INDEX_STATISTICS
Processing object type DATABASE_EXPORT/SCHEMA/VIEW/VIEW
Processing object type DATABASE_EXPORT/SCHEMA/VIEW/GRANT/OWNER_GRANT/OBJECT_GRANT
Processing object type DATABASE_EXPORT/SCHEMA/VIEW/COMMENT
Processing object type DATABASE_EXPORT/SCHEMA/PACKAGE_BODIES/PACKAGE/PACKAGE_BODY
Processing object type DATABASE_EXPORT/SCHEMA/TYPE/TYPE_BODY
Processing object type DATABASE_EXPORT/SCHEMA/TABLE/CONSTRAINT/REF_CONSTRAINT
Processing object type DATABASE_EXPORT/SCHEMA/TABLE/INDEX/BITMAP_INDEX/INDEX
Processing object type DATABASE_EXPORT/SCHEMA/TABLE/INDEX/STATISTICS/BITMAP_INDEX/INDEX_STATISTICS
Processing object type DATABASE_EXPORT/SCHEMA/TABLE/STATISTICS/TABLE_STATISTICS
Processing object type DATABASE_EXPORT/SCHEMA/TABLE/INDEX/DOMAIN_INDEX/INDEX
Processing object type DATABASE_EXPORT/SCHEMA/TABLE/POST_TABLE_ACTION
Processing object type DATABASE_EXPORT/SCHEMA/TABLE/TRIGGER
Processing object type DATABASE_EXPORT/SCHEMA/VIEW/TRIGGER
Processing object type DATABASE_EXPORT/SCHEMA/EVENT/TRIGGER
Processing object type DATABASE_EXPORT/SCHEMA/MATERIALIZED_VIEW
Processing object type DATABASE_EXPORT/SCHEMA/JOB
Processing object type DATABASE_EXPORT/SCHEMA/DIMENSION
Processing object type DATABASE_EXPORT/SCHEMA/TABLE/POST_INSTANCE/PROCACT_INSTANCE
Processing object type DATABASE_EXPORT/SCHEMA/TABLE/POST_INSTANCE/PROCDEPOBJ
Processing object type DATABASE_EXPORT/SCHEMA/POST_SCHEMA/PROCOBJ
Processing object type DATABASE_EXPORT/SCHEMA/POST_SCHEMA/PROCACT_SCHEMA
Processing object type DATABASE_EXPORT/AUDIT
. . exported "SH"."CUSTOMERS"                            9.853 MB   55500 rows
. . exported "PM"."ONLINE_MEDIA"                         7.854 MB       9 rows
. . exported "APEX_030200"."WWV_FLOW_PAGE_PLUGS"         5.205 MB    7416 rows
. . exported "SH"."COSTS":"COSTS_Q1_1998"                139.5 KB    4411 rows
. . exported "SH"."COSTS":"COSTS_Q1_1999"                183.5 KB    5884 rows
. . exported "SH"."COSTS":"COSTS_Q1_2000"                120.6 KB    3772 rows
. . exported "SH"."COSTS":"COSTS_Q1_2001"                227.8 KB    7328 rows
. . exported "SH"."COSTS":"COSTS_Q2_1998"                79.52 KB    2397 rows
. . exported "SH"."COSTS":"COSTS_Q2_1999"                132.5 KB    4179 rows
. . exported "SH"."COSTS":"COSTS_Q2_2000"                119.0 KB    3715 rows
. . exported "SH"."COSTS":"COSTS_Q2_2001"                184.5 KB    5882 rows
. . exported "SH"."COSTS":"COSTS_Q3_1998"                131.1 KB    4129 rows
. . exported "SH"."COSTS":"COSTS_Q3_1999"                137.3 KB    4336 rows
. . exported "SH"."COSTS":"COSTS_Q3_2000"                151.4 KB    4798 rows
. . exported "SH"."COSTS":"COSTS_Q3_2001"                234.4 KB    7545 rows
. . exported "SH"."COSTS":"COSTS_Q4_1998"                144.7 KB    4577 rows
. . exported "SH"."COSTS":"COSTS_Q4_1999"                159.0 KB    5060 rows
. . exported "SH"."COSTS":"COSTS_Q4_2000"                160.2 KB    5088 rows
. . exported "SH"."COSTS":"COSTS_Q4_2001"                278.4 KB    9011 rows
. . exported "SH"."SALES":"SALES_Q1_1998"                1.412 MB   43687 rows
. . exported "SH"."SALES":"SALES_Q1_1999"                2.071 MB   64186 rows
. . exported "SH"."SALES":"SALES_Q1_2000"                2.012 MB   62197 rows
. . exported "SH"."SALES":"SALES_Q1_2001"                1.965 MB   60608 rows
. . exported "SH"."SALES":"SALES_Q2_1998"                1.160 MB   35758 rows
. . exported "SH"."SALES":"SALES_Q2_1999"                1.754 MB   54233 rows
. . exported "SH"."SALES":"SALES_Q2_2000"                1.802 MB   55515 rows
. . exported "SH"."SALES":"SALES_Q2_2001"                2.051 MB   63292 rows
. . exported "SH"."SALES":"SALES_Q3_1998"                1.633 MB   50515 rows
. . exported "SH"."SALES":"SALES_Q3_1999"                2.166 MB   67138 rows
. . exported "SH"."SALES":"SALES_Q3_2000"                1.909 MB   58950 rows
. . exported "SH"."SALES":"SALES_Q3_2001"                2.130 MB   65769 rows
. . exported "SH"."SALES":"SALES_Q4_1998"                1.581 MB   48874 rows
. . exported "SH"."SALES":"SALES_Q4_1999"                2.014 MB   62388 rows
. . exported "SH"."SALES":"SALES_Q4_2000"                1.814 MB   55984 rows
. . exported "SH"."SALES":"SALES_Q4_2001"                2.257 MB   69749 rows
. . exported "APEX_030200"."WWV_FLOW_STEP_ITEMS"         3.505 MB    9671 rows
. . exported "APEX_030200"."WWV_FLOW_STEP_PROCESSING"    2.188 MB    2238 rows
. . exported "SYSMAN"."MGMT_MESSAGES"                    4.156 MB   23311 rows
. . exported "APEX_030200"."WWV_FLOW_DICTIONARY$"        2.909 MB   70601 rows
. . exported "SH"."SUPPLEMENTARY_DEMOGRAPHICS"           697.3 KB    4500 rows
. . exported "OE"."PRODUCT_DESCRIPTIONS"                 2.379 MB    8640 rows
. . exported "SYSMAN"."MGMT_ESA_REPORT"                  2.172 MB   31659 rows
. . exported "SYSMAN"."MGMT_HC_OS_COMPONENTS"            2.365 MB   23919 rows
. . exported "SYSMAN"."MGMT_HC_VENDOR_SW_COMPONENTS"     2.365 MB   23919 rows
. . exported "SYSMAN"."MGMT_HC_VENDOR_SW_SUMMARY"        1.844 MB   23919 rows
. . exported "APEX_030200"."WWV_FLOW_REGION_REPORT_COLUMN"  1.199 MB    7903 rows
. . exported "APEX_030200"."WWV_FLOW_STEP_ITEM_HELP"     1003. KB    6335 rows
. . exported "SYSMAN"."MGMT_HC_OS_PROPERTIES"            1.032 MB   15970 rows
. . exported "SYSMAN"."MGMT_METRICS"                     3.230 MB   12637 rows
. . exported "SYSMAN"."MGMT_IP_REPORT_ELEM_PARAMS"       756.8 KB    1788 rows
. . exported "APEX_030200"."WWV_FLOW_STEPS"              787.3 KB    1754 rows
. . exported "ORDDATA"."ORDDCM_DOCS"                     177.5 KB       9 rows
. . exported "SYSMAN"."MGMT_METRICS_RAW"                 899.0 KB   15303 rows
. . exported "SYSMAN"."MGMT_METRICS_1HOUR"               791.4 KB   10451 rows
. . exported "SYSMAN"."MGMT_INV_COMPONENT"               701.0 KB    2856 rows
. . exported "APEX_030200"."WWV_FLOW_LIST_ITEMS"         590.3 KB    3048 rows
. . exported "APEX_030200"."WWV_FLOW_STEP_VALIDATIONS"   611.4 KB    1990 rows
. . exported "APEX_030200"."WWV_FLOW_LIST_TEMPLATES"     111.3 KB     105 rows
. . exported "APEX_030200"."WWV_FLOW_STEP_BRANCHES"      508.6 KB    3255 rows
. . exported "APEX_030200"."WWV_FLOW_STEP_BUTTONS"       472.4 KB    3513 rows
. . exported "APEX_030200"."WWV_FLOW_TEMPLATES"          296.3 KB      64 rows
. . exported "PM"."PRINT_MEDIA"                          190.3 KB       4 rows
ORA-39181: Only partial table data may be exported due to fine grain access control on "OE"."PURCHASEORDER"
. . exported "OE"."PURCHASEORDER"                        243.9 KB     132 rows
. . exported "SH"."FWEEK_PSCAT_SALES_MV"                 419.8 KB   11266 rows
. . exported "SYSMAN"."MGMT_DB_INIT_PARAMS_ECM"          470.2 KB    8454 rows
. . exported "SYSMAN"."MGMT_JOB_STEP_PARAMS"             378.8 KB    4032 rows
. . exported "SYSMAN"."MGMT_POLICIES"                    714.3 KB    3260 rows
. . exported "APEX_030200"."WWV_FLOW_LIST_OF_VALUES_DATA"  392.0 KB    4184 rows
. . exported "SH"."PROMOTIONS"                           58.89 KB     503 rows
. . exported "SH"."TIMES"                                380.8 KB    1826 rows
. . exported "APEX_030200"."WWV_FLOW_MESSAGES$"          348.4 KB    3706 rows
. . exported "APEX_030200"."WWV_FLOW_PAGE_PLUG_TEMPLATES"  165.8 KB     166 rows
. . exported "APEX_030200"."WWV_FLOW_WORKSHEETS"         233.3 KB      30 rows
. . exported "SYSMAN"."ESM_COLLECTION"                   310.7 KB    4175 rows
. . exported "SYSMAN"."MGMT_DB_RECSEGMENTSETTINGS_ECM"   337.1 KB    4400 rows
. . exported "SYSMAN"."MGMT_METRICS_1DAY"                313.2 KB    3878 rows
. . exported "APEX_030200"."WWV_FLOW_CUSTOM_AUTH_SETUPS"  21.56 KB      11 rows
. . exported "APEX_030200"."WWV_FLOW_ROW_TEMPLATES"      81.09 KB      54 rows
. . exported "APEX_030200"."WWV_FLOW_STEP_COMPUTATIONS"  258.3 KB     984 rows
. . exported "SYSMAN"."MGMT_JOB_CRED_PARAMS"             30.90 KB      86 rows
. . exported "SYSMAN"."MGMT_VIOLATIONS"                  271.4 KB    1114 rows
. . exported "APEX_030200"."WWV_FLOW_LISTS_OF_VALUES$"   207.8 KB     959 rows
. . exported "OE"."WAREHOUSES"                           12.45 KB       9 rows
. . exported "SYSMAN"."MGMT_IP_SQL_STATEMENTS"           120.2 KB      31 rows
. . exported "SYSMAN"."MGMT_SEC_INFO"                    9.078 KB       1 rows
. . exported "APEX_030200"."WWV_FLOW_MENU_OPTIONS"       170.1 KB    1452 rows
. . exported "APEX_030200"."WWV_FLOW_WORKSHEET_COLUMNS"  185.7 KB     721 rows
. . exported "OE"."CUSTOMERS"                            77.97 KB     319 rows
. . exported "ORDDATA"."ORDDCM_STD_ATTRS"                188.8 KB    2415 rows
. . exported "SYSMAN"."MGMT_INV_DEPENDENCY_RULE"         195.9 KB    5124 rows
. . exported "SYSMAN"."MGMT_JOB_EXECPLAN"                199.4 KB    1448 rows
. . exported "SYSMAN"."MGMT_JOB_PROP_PARAMS"             10.82 KB      25 rows
. . exported "SYSMAN"."MGMT_POLICY_ASSOC"                97.54 KB    1208 rows
. . exported "SYSMAN"."MGMT_POLICY_ASSOC_CFG"            193.0 KB    1258 rows
. . exported "APEX_030200"."WWV_FLOW_PROCESSING"         73.33 KB      45 rows
. . exported "APEX_030200"."WWV_FLOW_RANDOM_IMAGES"      50.37 KB      42 rows
. . exported "ORDDATA"."ORDDCM_CT_PRED_OPRD"             13.84 KB      53 rows
. . exported "ORDDATA"."ORDDCM_MAPPING_DOCS"             7.890 KB       1 rows
. . exported "PM"."TEXTDOCS_NESTEDTAB"                   87.73 KB      12 rows
. . exported "SYSMAN"."MGMT_COLLECTIONS"                 34.78 KB     221 rows
. . exported "SYSMAN"."MGMT_COLLECTION_TASKS"            20.84 KB      28 rows
. . exported "SYSMAN"."MGMT_ECM_MD_ALL_TBL_COLUMNS"      110.2 KB     704 rows
. . exported "SYSMAN"."MGMT_IP_ELEM_DEFAULT_PARAMS"      42.48 KB     130 rows
. . exported "SYSMAN"."MGMT_JOB_PARAM_SOURCE"            79.73 KB     527 rows
. . exported "SYSMAN"."MGMT_JOB_SCHEDULE"                11.13 KB       2 rows
. . exported "SYSMAN"."MGMT_JOB_SEC_INFO"                    9 KB       7 rows
. . exported "SYSMAN"."MGMT_JOB_USER_PARAMS"             7.515 KB      15 rows
. . exported "SYSMAN"."MGMT_POLICY_ASSOC_CFG_PARAMS"     67.34 KB     802 rows
. . exported "SYSMAN"."MGMT_SYSTEM_PERFORMANCE_LOG"      92.35 KB    1205 rows
. . exported "APEX_030200"."WWV_FLOW_BANNER"             12.64 KB      10 rows
. . exported "APEX_030200"."WWV_FLOW_BUTTON_TEMPLATES"   17.35 KB      12 rows
. . exported "APEX_030200"."WWV_FLOW_FLASH_CHARTS"       30.08 KB       5 rows
. . exported "APEX_030200"."WWV_FLOW_FLASH_CHART_SERIES"  16.32 KB       5 rows
. . exported "APEX_030200"."WWV_FLOW_INSTALL"            36.28 KB       2 rows
. . exported "APEX_030200"."WWV_FLOW_LISTS"              51.74 KB     601 rows
. . exported "APEX_030200"."WWV_FLOW_PAGE_GENERIC_ATTR"  8.632 KB      44 rows
. . exported "APEX_030200"."WWV_FLOW_SHORTCUTS"          38.79 KB      39 rows
. . exported "APEX_030200"."WWV_FLOW_THEMES"             20.89 KB      10 rows
. . exported "OE"."PRODUCT_INFORMATION"                  72.77 KB     288 rows
. . exported "SYSMAN"."MGMT_AGENT_SEC_INFO"              7.960 KB       1 rows
. . exported "SYSMAN"."MGMT_ARU_PRODUCT_RELEASE_MAP"     84.79 KB    5956 rows
. . exported "SYSMAN"."MGMT_CATEGORY_MAP"                46.76 KB     637 rows
. . exported "SYSMAN"."MGMT_COLL_ITEM_METRICS"           20.57 KB     238 rows
. . exported "SYSMAN"."MGMT_CURRENT_METRICS"             38.60 KB     498 rows
. . exported "SYSMAN"."MGMT_CURRENT_VIOLATION"           46.64 KB     122 rows
. . exported "SYSMAN"."MGMT_ECM_SNAPSHOT_MD_COLUMNS"     87.19 KB     839 rows
. . exported "SYSMAN"."MGMT_JOB_COMMAND_BLOCK_PROCS"     5.914 KB       3 rows
. . exported "SYSMAN"."MGMT_JOB_LARGE_PARAMS"            5.929 KB       2 rows
. . exported "SYSMAN"."MGMT_JOB_OUTPUT"                  12.39 KB      11 rows
. . exported "SYSMAN"."MGMT_JOB_PARAMETER"               8.859 KB       2 rows
. . exported "SYSMAN"."MGMT_JOB_SQL_PARAMS"              6.828 KB       7 rows
. . exported "SYSMAN"."MGMT_JOB_SUBST_PARAMS"            6.296 KB      13 rows
. . exported "SYSMAN"."PARAM_VALUES_TAB"                 21.74 KB     240 rows
. . exported "SYSMAN"."MGMT_LAST_VIOLATION"              57.34 KB     795 rows
. . exported "SYSMAN"."MGMT_LICENSE_DEFINITIONS"         54.65 KB      59 rows
. . exported "SYSMAN"."MGMT_LONG_TEXT"                   7.148 KB      19 rows
. . exported "SYSMAN"."MGMT_METRIC_ERRORS"               42.70 KB     279 rows
. . exported "SYSMAN"."MGMT_POLICY_ASSOC_EVAL_DETAILS"   59.38 KB     540 rows
. . exported "SYSMAN"."MGMT_POLICY_VIOL_CTXT_DEF"        68.09 KB     642 rows
. . exported "SYSMAN"."MGMT_STORAGE_REPORT_DATA"         53.82 KB     240 rows
. . exported "SYSMAN"."MGMT_TASK_QTABLE"                 21.75 KB      28 rows
. . exported "APEX_030200"."WWV_COLUMN_EXCEPTIONS"       5.968 KB       3 rows
. . exported "APEX_030200"."WWV_FLOWS"                   59.70 KB      10 rows
. . exported "APEX_030200"."WWV_FLOW_ACTIVITY_LOG_NUMBER$"  5.468 KB       1 rows
. . exported "APEX_030200"."WWV_FLOW_ALT_CONFIG_PICK"    8.203 KB      37 rows
. . exported "APEX_030200"."WWV_FLOW_APPLICATION_GROUPS"  7.960 KB       1 rows
. . exported "APEX_030200"."WWV_FLOW_CALS"               18.85 KB      11 rows
. . exported "APEX_030200"."WWV_FLOW_CAL_TEMPLATES"      38.88 KB       9 rows
. . exported "APEX_030200"."WWV_FLOW_CHARSETS"           7.781 KB      32 rows
. . exported "APEX_030200"."WWV_FLOW_CLICKTHRU_LOG_NUMBER$"  5.882 KB       1 rows
. . exported "APEX_030200"."WWV_FLOW_COMPANIES"          8.835 KB       3 rows
. . exported "APEX_030200"."WWV_FLOW_COMPANY_SCHEMAS"    6.273 KB       1 rows
. . exported "APEX_030200"."WWV_FLOW_COMPANY_TYPES"      6.593 KB      32 rows
. . exported "APEX_030200"."WWV_FLOW_COMPUTATIONS"       14.92 KB      14 rows
. . exported "APEX_030200"."WWV_FLOW_COUNTRIES"          9.820 KB     240 rows
. . exported "APEX_030200"."WWV_FLOW_DB_AUTH"            5.460 KB       1 rows
. . exported "APEX_030200"."WWV_FLOW_DEVELOPER_ROLES"    5.921 KB       9 rows
. . exported "APEX_030200"."WWV_FLOW_DUAL100"            5.703 KB     100 rows
. . exported "APEX_030200"."WWV_FLOW_FIELD_TEMPLATES"    19.31 KB      36 rows
. . exported "APEX_030200"."WWV_FLOW_FND_USER"           23.80 KB       1 rows
. . exported "APEX_030200"."WWV_FLOW_HNT_COLUMN_INFO"    23.54 KB      58 rows
. . exported "APEX_030200"."WWV_FLOW_HNT_TABLE_INFO"     9.375 KB       8 rows
. . exported "APEX_030200"."WWV_FLOW_ICON_BAR"           19.25 KB      12 rows
. . exported "APEX_030200"."WWV_FLOW_ITEMS"              15.33 KB      89 rows
. . exported "APEX_030200"."WWV_FLOW_LANGUAGES"          16.15 KB     132 rows
. . exported "APEX_030200"."WWV_FLOW_LANGUAGE_MAP"       15.28 KB      90 rows
. . exported "APEX_030200"."WWV_FLOW_LOV_VALUES"         8.859 KB       6 rows
. . exported "APEX_030200"."WWV_FLOW_MENUS"              7.718 KB       7 rows
. . exported "APEX_030200"."WWV_FLOW_MENU_TEMPLATES"     14.14 KB       8 rows
. . exported "APEX_030200"."WWV_FLOW_PAGE_GROUPS"        14.28 KB     105 rows
. . exported "APEX_030200"."WWV_FLOW_PASSWORD_HISTORY"   6.671 KB       1 rows
. . exported "APEX_030200"."WWV_FLOW_PATCHES"            10.68 KB       9 rows
. . exported "APEX_030200"."WWV_FLOW_PICK_END_USERS"     6.015 KB       4 rows
. . exported "APEX_030200"."WWV_FLOW_PICK_PAGE_VIEWS"    6.093 KB       5 rows
. . exported "APEX_030200"."WWV_FLOW_PLATFORM_PREFS"     9.007 KB      21 rows
. . exported "APEX_030200"."WWV_FLOW_POPUP_LOV_TEMPLATE"  32.22 KB      10 rows
. . exported "APEX_030200"."WWV_FLOW_QUERY_COLUMN"       9.921 KB      18 rows
. . exported "APEX_030200"."WWV_FLOW_QUERY_CONDITION"    9.601 KB       6 rows
. . exported "APEX_030200"."WWV_FLOW_QUERY_DEFINITION"   7.640 KB       6 rows
. . exported "APEX_030200"."WWV_FLOW_QUERY_OBJECT"       8.304 KB       6 rows
. . exported "APEX_030200"."WWV_FLOW_REGION_UPD_RPT_COLS"  36.80 KB     439 rows
. . exported "APEX_030200"."WWV_FLOW_RESTRICTED_SCHEMAS"  8.234 KB      46 rows
. . exported "APEX_030200"."WWV_FLOW_SECURITY_SCHEMES"   15.28 KB      19 rows
. . exported "APEX_030200"."WWV_FLOW_STANDARD_CSS"       9.742 KB      27 rows
. . exported "APEX_030200"."WWV_FLOW_STANDARD_ICONS"     16.92 KB     319 rows
. . exported "APEX_030200"."WWV_FLOW_STANDARD_JS"        7.203 KB       2 rows
. . exported "APEX_030200"."WWV_FLOW_SW_CREATE_KEYWORDS"  6.867 KB      10 rows
. . exported "APEX_030200"."WWV_FLOW_SW_MAIN_KEYWORDS"   10.85 KB     199 rows
. . exported "APEX_030200"."WWV_FLOW_SW_SET_KEYWORDS"     6.75 KB       4 rows
. . exported "APEX_030200"."WWV_FLOW_SW_SQLPLUS_CMD"     5.101 KB       8 rows
. . exported "APEX_030200"."WWV_FLOW_TABS"               13.52 KB       3 rows
. . exported "APEX_030200"."WWV_FLOW_TOPLEVEL_TABS"      13.17 KB       5 rows
. . exported "APEX_030200"."WWV_FLOW_TRANSLATABLE_COLS$"  29.54 KB     232 rows
. . exported "APEX_030200"."WWV_FLOW_TREES"              25.79 KB       3 rows
. . exported "APEX_030200"."WWV_FLOW_UPGRADE_PROGRESS"   21.89 KB      89 rows
. . exported "APEX_030200"."WWV_FLOW_UPG_TAB_NAME_CHANGES"  8.773 KB      42 rows
. . exported "APEX_030200"."WWV_FLOW_UPG_TAB_OBSOLETE"   6.468 KB      17 rows
. . exported "APEX_030200"."WWV_FLOW_USER_ACCESS_LOG_NUM$"  5.882 KB       1 rows
. . exported "APEX_030200"."WWV_FLOW_WORKSHEET_RPTS"     37.01 KB      30 rows
. . exported "APEX_030200"."WWV_FLOW_WORKSPACE_REQ_SIZE"  6.851 KB      14 rows
. . exported "APEX_030200"."WWV_MIG_EXPORTER"            7.382 KB       5 rows
. . exported "APEX_030200"."WWV_MIG_FRM_OLB_XMLTAGTABLEMAP"  11.28 KB      45 rows
. . exported "APEX_030200"."WWV_MIG_FRM_XMLTAGTABLEMAP"  10.54 KB      36 rows
. . exported "APEX_030200"."WWV_MIG_MENU_XMLTAGTABLEMAP"  8.703 KB       7 rows
. . exported "APEX_030200"."WWV_MIG_RESERVED_WORDS"      5.890 KB      87 rows
. . exported "APEX_030200"."WWV_MIG_RPT_XMLTAGTABLEMAP"  9.132 KB      15 rows
. . exported "HR"."COUNTRIES"                            6.367 KB      25 rows
. . exported "HR"."DEPARTMENTS"                          7.007 KB      27 rows
. . exported "HR"."EMPLOYEES"                            16.80 KB     107 rows
. . exported "HR"."JOBS"                                 6.992 KB      19 rows
. . exported "HR"."JOB_HISTORY"                          7.054 KB      10 rows
. . exported "HR"."LOCATIONS"                            8.273 KB      23 rows
. . exported "HR"."REGIONS"                              5.476 KB       4 rows
. . exported "IX"."AQ$_ORDERS_QUEUETABLE_S"              10.91 KB       4 rows
. . exported "IX"."AQ$_STREAMS_QUEUE_TABLE_S"            11.17 KB       1 rows
. . exported "OE"."CATEGORIES_TAB"                       14.15 KB      22 rows
. . exported "OE"."SUBCATEGORY_REF_LIST_NESTEDTAB"       6.585 KB      21 rows
. . exported "OE"."PRODUCT_REF_LIST_NESTEDTAB"           12.50 KB     288 rows
. . exported "OE"."INVENTORIES"                          21.67 KB    1112 rows
. . exported "OE"."ORDERS"                               12.38 KB     105 rows
. . exported "OE"."ORDER_ITEMS"                          20.87 KB     665 rows
. . exported "OE"."PROMOTIONS"                             5.5 KB       2 rows
. . exported "ORDDATA"."ORDDCM_ANON_ACTION_TYPES"        5.484 KB       4 rows
. . exported "ORDDATA"."ORDDCM_ANON_ATTRS"               8.703 KB      37 rows
. . exported "ORDDATA"."ORDDCM_ANON_RULES"               6.289 KB       3 rows
. . exported "ORDDATA"."ORDDCM_ANON_RULE_TYPES"          5.546 KB       3 rows
. . exported "ORDDATA"."ORDDCM_CT_ACTION"                6.664 KB       7 rows
. . exported "ORDDATA"."ORDDCM_CT_DAREFS"                6.187 KB      72 rows
. . exported "ORDDATA"."ORDDCM_CT_LOCATORPATHS"          10.07 KB      95 rows
. . exported "ORDDATA"."ORDDCM_CT_MACRO_DEP"             5.460 KB       1 rows
. . exported "ORDDATA"."ORDDCM_CT_MACRO_PAR"             5.476 KB       2 rows
. . exported "ORDDATA"."ORDDCM_CT_PRED"                  8.320 KB      61 rows
. . exported "ORDDATA"."ORDDCM_CT_PRED_PAR"              5.937 KB       3 rows
. . exported "ORDDATA"."ORDDCM_CT_PRED_SET"              8.906 KB       9 rows
. . exported "ORDDATA"."ORDDCM_DATA_MODEL"               5.859 KB       1 rows
. . exported "ORDDATA"."ORDDCM_DICT_ATTRS"               33.95 KB    2418 rows
. . exported "ORDDATA"."ORDDCM_DOC_REFS"                   6.5 KB       7 rows
. . exported "ORDDATA"."ORDDCM_DOC_TYPES"                6.976 KB       8 rows
. . exported "ORDDATA"."ORDDCM_INSTALL_DOCS"             6.539 KB       9 rows
. . exported "ORDDATA"."ORDDCM_INTERNAL_TAGS"            6.210 KB      42 rows
. . exported "ORDDATA"."ORDDCM_PREFS_VALID_VALUES_TAB"   6.492 KB      33 rows
. . exported "ORDDATA"."ORDDCM_PREFS_LOOKUP"             9.789 KB      13 rows
. . exported "ORDDATA"."ORDDCM_PREFS_DEF_VALUES_TAB"     5.679 KB       7 rows
. . exported "ORDDATA"."ORDDCM_PRV_ATTRS"                9.804 KB       3 rows
. . exported "ORDDATA"."ORDDCM_RT_PREF_PARAMS"           8.375 KB      13 rows
. . exported "ORDDATA"."ORDDCM_UID_DEFS"                 33.49 KB     245 rows
. . exported "ORDDATA"."ORDDCM_VR_DT_MAP"                6.507 KB      32 rows
. . exported "SCOTT"."DEPT"                              5.929 KB       4 rows
. . exported "SCOTT"."EMP"                               8.562 KB      14 rows
. . exported "SCOTT"."SALGRADE"                          5.859 KB       5 rows
. . exported "SCOTT"."SCOTT_TABLE"                       5.468 KB       4 rows
. . exported "SCOTT"."TEST"                              5.062 KB       5 rows
. . exported "SCOTT"."TEST_DUMP"                         5.156 KB      20 rows
. . exported "SH"."CAL_MONTH_SALES_MV"                   6.312 KB      48 rows
. . exported "SH"."CHANNELS"                              7.25 KB       5 rows
. . exported "SH"."COUNTRIES"                            10.20 KB      23 rows
. . exported "SH"."PRODUCTS"                             26.17 KB      72 rows
. . exported "SYSMAN"."AQ$_MGMT_LOADER_QTABLE_S"         10.81 KB       2 rows
. . exported "SYSMAN"."AQ$_MGMT_NOTIFY_QTABLE_S"         10.81 KB       2 rows
. . exported "SYSMAN"."EMDW_TRACE_CONFIG"                7.054 KB       9 rows
. . exported "SYSMAN"."EM_PAGE_CONDITION_METADATA"       5.640 KB       7 rows
. . exported "SYSMAN"."EM_PAGE_CUST_METADATA"            6.031 KB       4 rows
. . exported "SYSMAN"."EUME2E_ASSOCS_LOOKUP"             6.601 KB       9 rows
. . exported "SYSMAN"."MGMT_ADMIN_LICENSES"              5.296 KB      19 rows
. . exported "SYSMAN"."MGMT_ALL_TARGET_PROPS"            9.773 KB       5 rows
. . exported "SYSMAN"."MGMT_ARU_FAMILY_PRODUCT_MAP"      25.85 KB    1660 rows
. . exported "SYSMAN"."MGMT_ARU_LANGUAGES"               6.156 KB      40 rows
. . exported "SYSMAN"."MGMT_ARU_OUI_COMPONENTS"          21.33 KB     393 rows
. . exported "SYSMAN"."MGMT_ARU_PLATFORMS"               9.578 KB      76 rows
. . exported "SYSMAN"."MGMT_ARU_PRODUCTS"                34.71 KB     744 rows
. . exported "SYSMAN"."MGMT_ARU_RELEASES"                19.89 KB     863 rows
. . exported "SYSMAN"."MGMT_AUDIT_DESTINATION"           5.492 KB       1 rows
. . exported "SYSMAN"."MGMT_AUDIT_MASTER"                5.070 KB       1 rows
. . exported "SYSMAN"."MGMT_AVAILABILITY"                13.10 KB     138 rows
. . exported "SYSMAN"."MGMT_AVAILABILITY_MARKER"         6.015 KB       5 rows
. . exported "SYSMAN"."MGMT_AVAILABLE_SEARCHES"          7.023 KB      15 rows
. . exported "SYSMAN"."MGMT_BLACKOUT_PROXY_TARGETS"      5.140 KB       5 rows
. . exported "SYSMAN"."MGMT_BLACKOUT_REASON"             9.617 KB      60 rows
. . exported "SYSMAN"."MGMT_BSLN_METRICS"                7.351 KB       6 rows
. . exported "SYSMAN"."MGMT_CALLBACKS"                   12.19 KB      87 rows
. . exported "SYSMAN"."MGMT_CATEGORIES"                  6.226 KB      12 rows
. . exported "SYSMAN"."MGMT_CATEGORY_CLASSES"            5.460 KB       2 rows
. . exported "SYSMAN"."MGMT_COLLECTION_METRIC_TASKS"      20.5 KB     209 rows
. . exported "SYSMAN"."MGMT_COLL_ITEMS"                  26.13 KB     264 rows
. . exported "SYSMAN"."MGMT_COLL_ITEM_PROPERTIES"        10.67 KB      48 rows
. . exported "SYSMAN"."MGMT_CORRECTIVE_ACTION"           6.734 KB       2 rows
. . exported "SYSMAN"."MGMT_CREATED_USERS"               5.976 KB       4 rows
. . exported "SYSMAN"."MGMT_CREDENTIALS2"                7.742 KB       3 rows
. . exported "SYSMAN"."MGMT_CREDENTIAL_SETS"             12.03 KB      35 rows
. . exported "SYSMAN"."MGMT_CREDENTIAL_SET_COLUMNS"      15.15 KB      86 rows
. . exported "SYSMAN"."MGMT_CREDENTIAL_SET_COL_VALS"     7.953 KB       8 rows
. . exported "SYSMAN"."MGMT_CREDENTIAL_TYPES"            8.625 KB      19 rows
. . exported "SYSMAN"."MGMT_CREDENTIAL_TYPE_COLUMNS"     12.24 KB      45 rows
. . exported "SYSMAN"."MGMT_CREDENTIAL_TYPE_COL_VALS"    7.648 KB       3 rows
. . exported "SYSMAN"."MGMT_CREDENTIAL_TYPE_REF"         8.476 KB       9 rows
. . exported "SYSMAN"."MGMT_CS_CONFIG_STANDARD"          15.92 KB       3 rows
. . exported "SYSMAN"."MGMT_CS_HIERARCHY"                11.29 KB     117 rows
. . exported "SYSMAN"."MGMT_CS_KEYWORD"                  5.554 KB       3 rows
. . exported "SYSMAN"."MGMT_CS_RULE"                     31.97 KB      94 rows
. . exported "SYSMAN"."MGMT_CS_RULEFOLDER"               9.328 KB      23 rows
. . exported "SYSMAN"."MGMT_CURRENT_AVAILABILITY"        6.437 KB       5 rows
. . exported "SYSMAN"."MGMT_CURRENT_METRIC_ERRORS"        7.75 KB       2 rows
. . exported "SYSMAN"."MGMT_DB_CONTROLFILES_ECM"         12.46 KB      44 rows
. . exported "SYSMAN"."MGMT_DB_DATAFILES_ECM"            23.84 KB     154 rows
. . exported "SYSMAN"."MGMT_DB_DBNINSTANCEINFO_ECM"      14.55 KB      24 rows
. . exported "SYSMAN"."MGMT_DB_FEATUREUSAGE"             33.64 KB     161 rows
. . exported "SYSMAN"."MGMT_DB_HDM_METRIC_HELPER"        6.687 KB       1 rows
. . exported "SYSMAN"."MGMT_DB_LICENSE_ECM"              7.812 KB      24 rows
. . exported "SYSMAN"."MGMT_DB_OPTIONS_ECM"               15.5 KB     264 rows
. . exported "SYSMAN"."MGMT_DB_RECTSSETTINGS_ECM"        7.523 KB      22 rows
. . exported "SYSMAN"."MGMT_DB_RECUSERSETTINGS_ECM"      7.101 KB      22 rows
. . exported "SYSMAN"."MGMT_DB_REDOLOGS_ECM"             16.07 KB      72 rows
. . exported "SYSMAN"."MGMT_DB_ROLLBACK_SEGS_ECM"        13.42 KB      24 rows
. . exported "SYSMAN"."MGMT_DB_SGA_ECM"                  14.60 KB     216 rows
. . exported "SYSMAN"."MGMT_DB_TABLESPACES_ECM"          29.10 KB     154 rows
. . exported "SYSMAN"."MGMT_DELTA_ENTRY"                 16.14 KB     116 rows
. . exported "SYSMAN"."MGMT_DELTA_ENTRY_VALUES"          33.67 KB     547 rows
. . exported "SYSMAN"."MGMT_DELTA_IDS"                   11.62 KB      45 rows
. . exported "SYSMAN"."MGMT_DELTA_ID_VALUES"             10.14 KB      92 rows
. . exported "SYSMAN"."MGMT_DELTA_SNAP"                  15.20 KB      43 rows
. . exported "SYSMAN"."MGMT_DEPLOYMENT_SECTIONS"           5.5 KB       1 rows
. . exported "SYSMAN"."MGMT_DM_ALITEMS"                  7.046 KB      48 rows
. . exported "SYSMAN"."MGMT_DM_RULEENTRY"                12.51 KB      45 rows
. . exported "SYSMAN"."MGMT_DM_RULETEMPLATES"            9.882 KB      19 rows
. . exported "SYSMAN"."MGMT_ECM_ARU_MAP"                 8.164 KB      29 rows
. . exported "SYSMAN"."MGMT_ECM_CSA_OUT_OF_BOX"          5.023 KB       1 rows
. . exported "SYSMAN"."MGMT_ECM_GEN_SNAPSHOT"            27.94 KB     146 rows
. . exported "SYSMAN"."MGMT_ECM_MD_HIST_TBLS"            19.63 KB      39 rows
. . exported "SYSMAN"."MGMT_ECM_RESOURCES"               6.804 KB      14 rows
. . exported "SYSMAN"."MGMT_ECM_SNAPSHOT"                11.25 KB      22 rows
. . exported "SYSMAN"."MGMT_ECM_SNAPSHOT_MD_TABLES"      21.80 KB     122 rows
. . exported "SYSMAN"."MGMT_ECM_SNAPSHOT_METADATA"       17.28 KB      53 rows
. . exported "SYSMAN"."MGMT_ECM_SNAP_COMPONENT_INFO"     16.71 KB      84 rows
. . exported "SYSMAN"."MGMT_EMD_PING"                    11.41 KB       1 rows
. . exported "SYSMAN"."MGMT_EMD_PING_CHECK"              5.460 KB       1 rows
. . exported "SYSMAN"."MGMT_ERROR_MASTER"                6.453 KB      12 rows
. . exported "SYSMAN"."MGMT_FAILOVER_CALLBACKS"          5.179 KB       4 rows
. . exported "SYSMAN"."MGMT_FAILOVER_TABLE"              6.765 KB       1 rows
. . exported "SYSMAN"."MGMT_FBP_PATCHING_GUIDS"          5.609 KB       3 rows
. . exported "SYSMAN"."MGMT_FLAT_TARGET_ASSOC"           7.828 KB       3 rows
. . exported "SYSMAN"."MGMT_GROUP_DEFAULT_CHART"         10.25 KB       9 rows
. . exported "SYSMAN"."MGMT_HA_INFO_ECM"                 8.796 KB      24 rows
. . exported "SYSMAN"."MGMT_HA_MTTR"                     5.859 KB       1 rows
. . exported "SYSMAN"."MGMT_HA_RMAN_CONFIG_ECM"          6.859 KB      24 rows
. . exported "SYSMAN"."MGMT_HC_CPU_DETAILS"              9.218 KB      21 rows
. . exported "SYSMAN"."MGMT_HC_HARDWARE_MASTER"          11.14 KB      21 rows
. . exported "SYSMAN"."MGMT_HC_NIC_DETAILS"              15.12 KB      63 rows
. . exported "SYSMAN"."MGMT_HC_OS_SUMMARY"               10.57 KB      21 rows
. . exported "SYSMAN"."MGMT_HC_SYSTEM_SUMMARY"           8.007 KB      21 rows
. . exported "SYSMAN"."MGMT_HTTP_SESSION_CALLBACKS"      5.492 KB       1 rows
. . exported "SYSMAN"."MGMT_INV_CONTAINER"               10.90 KB      42 rows
. . exported "SYSMAN"."MGMT_INV_CONTAINER_PROPERTY"      8.531 KB      63 rows
. . exported "SYSMAN"."MGMT_INV_SUMMARY"                 13.37 KB      42 rows
. . exported "SYSMAN"."MGMT_IP_ELEM_PARAM_CLASSES"       19.90 KB     116 rows
. . exported "SYSMAN"."MGMT_IP_ELEM_TARGET_TYPES"         9.25 KB      86 rows
. . exported "SYSMAN"."MGMT_IP_REPORT_DEF"               37.09 KB     105 rows
. . exported "SYSMAN"."MGMT_IP_REPORT_DEF_ELEMENTS"      38.97 KB     300 rows
. . exported "SYSMAN"."MGMT_IP_REPORT_DEF_JIT_TYPES"     8.406 KB      93 rows
. . exported "SYSMAN"."MGMT_IP_REPORT_ELEM_DEF"          20.96 KB      77 rows
. . exported "SYSMAN"."MGMT_JOB"                         14.44 KB       4 rows
. . exported "SYSMAN"."MGMT_JOB_CALLBACKS"               6.281 KB       6 rows
. . exported "SYSMAN"."MGMT_JOB_COMMAND"                 17.67 KB     136 rows
. . exported "SYSMAN"."MGMT_JOB_DISPLAY_ERROR_CODES"     6.765 KB       8 rows
. . exported "SYSMAN"."MGMT_JOB_EVENT"                   5.476 KB       1 rows
. . exported "SYSMAN"."MGMT_JOB_EXECUTION"               17.05 KB       8 rows
. . exported "SYSMAN"."MGMT_JOB_EXEC_SUMMARY"            16.80 KB      24 rows
. . exported "SYSMAN"."MGMT_JOB_HISTORY"                 21.59 KB      41 rows
. . exported "SYSMAN"."MGMT_JOB_LOCK_INFO"               8.257 KB       5 rows
. . exported "SYSMAN"."MGMT_JOB_LOCK_TARGETS"            7.078 KB       5 rows
. . exported "SYSMAN"."MGMT_JOB_PURGE_CRITERIA"          6.453 KB       2 rows
. . exported "SYSMAN"."MGMT_JOB_PURGE_POLICIES"          5.523 KB       3 rows
. . exported "SYSMAN"."MGMT_JOB_PURGE_VALUES"            5.984 KB       2 rows
. . exported "SYSMAN"."MGMT_JOB_SINGLE_TARGET_TYPES"     9.093 KB     120 rows
. . exported "SYSMAN"."MGMT_JOB_STATE_CHANGES"           14.06 KB      64 rows
. . exported "SYSMAN"."MGMT_JOB_TYPE_DISPLAY_INFO"       11.78 KB      83 rows
. . exported "SYSMAN"."MGMT_JOB_TYPE_INFO"               34.12 KB     155 rows
. . exported "SYSMAN"."MGMT_JOB_TYPE_MAX_VERSIONS"       13.26 KB     150 rows
. . exported "SYSMAN"."MGMT_JOB_TYPE_PARAM_DSPLY_INFO"   31.05 KB     304 rows
. . exported "SYSMAN"."MGMT_JOB_TYPE_PARAM_URI_INFO"     8.148 KB       7 rows
. . exported "SYSMAN"."MGMT_JOB_TYPE_URI_INFO"           11.16 KB      79 rows
. . exported "SYSMAN"."MGMT_JOB_VALUE_PARAMS"            12.15 KB     140 rows
. . exported "SYSMAN"."MGMT_LAST_SYNC_LOAD_DETAILS"      5.875 KB       1 rows
. . exported "SYSMAN"."MGMT_LICENSABLE_TARGET_TYPES"     7.117 KB      49 rows
. . exported "SYSMAN"."MGMT_LICENSED_TARGETS"            6.296 KB      10 rows
. . exported "SYSMAN"."MGMT_LOADER_DESIGNATORS"          5.773 KB      40 rows
. . exported "SYSMAN"."MGMT_LOGIN_ASSISTANTS"            6.046 KB       2 rows
. . exported "SYSMAN"."MGMT_MASTER_CHANGED_CALLBACK"     5.898 KB       1 rows
. . exported "SYSMAN"."MGMT_METADATA_SETS"               9.984 KB      76 rows
. . exported "SYSMAN"."MGMT_METRICS_COMPOSITE_KEYS"      37.71 KB     367 rows
. . exported "SYSMAN"."MGMT_METRICS_EXT"                 10.21 KB      20 rows
. . exported "SYSMAN"."MGMT_METRIC_DEPENDENCY_DEF"       8.062 KB      16 rows
. . exported "SYSMAN"."MGMT_METRIC_VERSIONS"             17.10 KB     273 rows
. . exported "SYSMAN"."MGMT_MP_HOMEPAGE_REPORTS"         7.789 KB      30 rows
. . exported "SYSMAN"."MGMT_NESTED_JOB_TARGETS"          12.01 KB      61 rows
. . exported "SYSMAN"."MGMT_NOTIFY_FORMAT_HANDLERS"      5.804 KB       6 rows
. . exported "SYSMAN"."MGMT_NOTIFY_JOB_RULE_CONFIGS"     10.22 KB       1 rows
. . exported "SYSMAN"."MGMT_NOTIFY_PROFILES"             6.718 KB       3 rows
. . exported "SYSMAN"."MGMT_NOTIFY_QUEUES"               6.226 KB      27 rows
. . exported "SYSMAN"."MGMT_NOTIFY_RULES"                8.320 KB       7 rows
. . exported "SYSMAN"."MGMT_NOTIFY_RULE_CONFIGS"         25.31 KB      41 rows
. . exported "SYSMAN"."MGMT_OMS_PARAMETERS"              10.64 KB      61 rows
. . exported "SYSMAN"."MGMT_OPERATIONS_MASTER"           13.85 KB      25 rows
. . exported "SYSMAN"."MGMT_OUI_ARU_MAP"                 5.804 KB      27 rows
. . exported "SYSMAN"."MGMT_PAF_APPLICATIONS"            11.32 KB       4 rows
. . exported "SYSMAN"."MGMT_PAF_JOBTYPES"                10.17 KB       2 rows
. . exported "SYSMAN"."MGMT_PAF_JOBTYPE_PARAMS"          14.94 KB      16 rows
. . exported "SYSMAN"."MGMT_PAF_PARAM_GROUPS"            10.79 KB       6 rows
. . exported "SYSMAN"."MGMT_PAF_PROCEDURES"              13.58 KB       2 rows
. . exported "SYSMAN"."MGMT_PAF_TEXTUAL_DATA"            24.24 KB       2 rows
. . exported "SYSMAN"."MGMT_PARAMETERS"                  13.78 KB      84 rows
. . exported "SYSMAN"."MGMT_PDP_COLUMN_METADATA"         5.968 KB       3 rows
. . exported "SYSMAN"."MGMT_PDP_METADATA"                5.906 KB       2 rows
. . exported "SYSMAN"."MGMT_PDP_PARAM_METADATA"          7.007 KB       9 rows
. . exported "SYSMAN"."MGMT_PDP_SETTING_METADATA"        7.929 KB       3 rows
. . exported "SYSMAN"."MGMT_PERFORMANCE_NAMES"           11.70 KB     106 rows
. . exported "SYSMAN"."MGMT_POLICY_ASSOC_EVAL_SUMM"      20.96 KB     167 rows
. . exported "SYSMAN"."MGMT_POLICY_BIND_VARS"            12.49 KB     169 rows
. . exported "SYSMAN"."MGMT_POLICY_PARAMETERS"           8.117 KB      27 rows
. . exported "SYSMAN"."MGMT_POLICY_TYPE_VERSIONS"        20.38 KB     585 rows
. . exported "SYSMAN"."MGMT_PRIVS"                       8.265 KB      24 rows
. . exported "SYSMAN"."MGMT_PRIV_GRANTS"                 12.69 KB     113 rows
. . exported "SYSMAN"."MGMT_PRIV_INCLUDES"               5.812 KB      13 rows
. . exported "SYSMAN"."MGMT_PROV_COLLECTION"             5.632 KB      21 rows
. . exported "SYSMAN"."MGMT_PROV_HARDWARE"               11.20 KB       1 rows
. . exported "SYSMAN"."MGMT_PROV_TGT_STATUS"               7.5 KB       1 rows
. . exported "SYSMAN"."MGMT_PURGE_POLICY"                10.57 KB      16 rows
. . exported "SYSMAN"."MGMT_PURGE_POLICY_GROUP"          6.140 KB       7 rows
. . exported "SYSMAN"."MGMT_PURGE_POLICY_TARGET_STATE"   10.52 KB      36 rows
. . exported "SYSMAN"."MGMT_REBUILD_INDEXES"             5.859 KB       1 rows
. . exported "SYSMAN"."MGMT_ROLES"                       5.429 KB       1 rows
. . exported "SYSMAN"."MGMT_ROLE_GRANTS"                 6.343 KB       1 rows
. . exported "SYSMAN"."MGMT_ROWSET_HANDLERS"             7.195 KB      14 rows
. . exported "SYSMAN"."MGMT_RT_BOOTSTRAP_TIMES"          5.578 KB       5 rows
. . exported "SYSMAN"."MGMT_SNAPSHOT_METRIC_MAP"         19.13 KB     221 rows
. . exported "SYSMAN"."MGMT_SPACE_METRICS"               8.406 KB       1 rows
. . exported "SYSMAN"."MGMT_STORAGE_REPORT_ALIAS"        22.09 KB     240 rows
. . exported "SYSMAN"."MGMT_STORAGE_REPORT_ISSUES"       17.25 KB     100 rows
. . exported "SYSMAN"."MGMT_STORAGE_REPORT_KEYS"         21.83 KB     260 rows
. . exported "SYSMAN"."MGMT_STRING_METRIC_HISTORY"       12.94 KB      85 rows
. . exported "SYSMAN"."MGMT_SWLIB_DIRECTORIES"           7.398 KB       5 rows
. . exported "SYSMAN"."MGMT_SYSTEM_ERROR_LOG"            9.179 KB       3 rows
. . exported "SYSMAN"."MGMT_TARGETS"                     16.54 KB       5 rows
. . exported "SYSMAN"."MGMT_TARGET_ADD_CALLBACKS"        5.976 KB      13 rows
. . exported "SYSMAN"."MGMT_TARGET_ASSOCS"               7.539 KB       6 rows
. . exported "SYSMAN"."MGMT_TARGET_ASSOC_DEFS"           11.03 KB      15 rows
. . exported "SYSMAN"."MGMT_TARGET_ASSOC_ERROR"          6.484 KB       4 rows
. . exported "SYSMAN"."MGMT_TARGET_ASSOC_STATUS"         5.953 KB       4 rows
. . exported "SYSMAN"."MGMT_TARGET_CREDENTIALS"          6.710 KB       1 rows
. . exported "SYSMAN"."MGMT_TARGET_DELETE_EXCEPTIONS"    5.945 KB      35 rows
. . exported "SYSMAN"."MGMT_TARGET_PROPERTIES"           17.14 KB     204 rows
. . exported "SYSMAN"."MGMT_TARGET_PROP_DEFS"            38.64 KB     288 rows
. . exported "SYSMAN"."MGMT_TARGET_ROLLUP_TIMES"         12.26 KB     130 rows
. . exported "SYSMAN"."MGMT_TARGET_TYPES"                9.718 KB      28 rows
. . exported "SYSMAN"."MGMT_TARGET_TYPE_COMPONENT_MAP"   6.453 KB       5 rows
. . exported "SYSMAN"."MGMT_TARGET_TYPE_VERSIONS"        11.92 KB      37 rows
. . exported "SYSMAN"."MGMT_TASK_WORKER_COUNTS"          5.507 KB       2 rows
. . exported "SYSMAN"."MGMT_TYPE_PROPERTIES"             8.578 KB      75 rows
. . exported "SYSMAN"."MGMT_USER_CALLBACKS"                6.5 KB      27 rows
. . exported "SYSMAN"."MGMT_USER_CAS"                    5.460 KB       2 rows
. . exported "SYSMAN"."MGMT_USER_CONTEXT"                5.898 KB       4 rows
. . exported "SYSMAN"."MGMT_USER_FOLDERS"                6.835 KB      24 rows
. . exported "SYSMAN"."MGMT_USER_SUBTAB_COL_PREFS"       8.687 KB      41 rows
. . exported "SYSMAN"."MGMT_VERSIONS"                    6.835 KB       5 rows
. . exported "SYSMAN"."MGMT_VIEW_USER_CREDENTIALS"       5.515 KB       1 rows
. . exported "SYSMAN"."MGMT_VIOLATION_CONTEXT"           32.64 KB     233 rows
. . exported "SYSTEM"."REPCAT$_AUDIT_ATTRIBUTE"          6.328 KB       2 rows
. . exported "SYSTEM"."REPCAT$_OBJECT_TYPES"             6.882 KB      28 rows
. . exported "SYSTEM"."REPCAT$_RESOLUTION_METHOD"        5.835 KB      19 rows
. . exported "SYSTEM"."REPCAT$_TEMPLATE_STATUS"          5.484 KB       3 rows
. . exported "SYSTEM"."REPCAT$_TEMPLATE_TYPES"           6.289 KB       2 rows
. . exported "APEX_030200"."WWV_FLOWS_RESERVED"              0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_ACTIVITY_LOG1$"         0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_ACTIVITY_LOG2$"         0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_ALTERNATE_CONFIG"       0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_ALT_CONFIG_DETAIL"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_APP_BUILD_PREF"         0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_APP_COMMENTS"           0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_BUILDER_AUDIT_TRAIL"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_CLICKTHRU_LOG$"         0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_CLICKTHRU_LOG2$"        0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_COLLECTIONS$"           0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_COLLECTION_MEMBERS$"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_COMPOUND_CONDITIONS"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_CSS_REPOSITORY"         0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_CUSTOMIZED_TASKS"       0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_DATA"                   0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_DATA_LOAD_BAD_LOG"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_DATA_LOAD_UNLOAD"       0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_DEBUG"                  0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_DEVELOPERS"             0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_DYNAMIC_TRANSLATIONS$"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_EFFECTIVE_USERID_MAP"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_ENTRY_POINTS"           0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_ENTRY_POINT_ARGS"       0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_FILE_OBJECTS$PART"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_FND_GROUP_USERS"        0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_FND_USER_GROUPS"        0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_FOLDERS"                0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_HNT_ARGUMENT_INFO"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_HNT_LOV_DATA"           0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_HNT_PROCEDURE_INFO"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_HTML_REPOSITORY"        0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_ICON_BAR_ATTRIBUTES"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_IMAGE_REPOSITORY"       0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_IMPORT_EXPORT"          0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_INSTALL_BUILD_OPT"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_INSTALL_CHECKS"         0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_INSTALL_SCRIPTS"        0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_JOBS"                   0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_JOB_BIND_VALUES"        0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_LOCK_PAGE"              0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_LOCK_PAGE_LOG"          0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_MAIL_ATTACHMENTS"       0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_MAIL_LOG"               0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_MAIL_QUEUE"             0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_MODELS"                 0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_MODEL_PAGES"            0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_MODEL_PAGE_COLS"        0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_MODEL_PAGE_REGIONS"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_ONLINE_HELP"            0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_ONLINE_HELP_JA"         0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_PAGES_RESERVED"         0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_PAGE_CACHE"             0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_PAGE_SUBMISSIONS"       0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_PICK_DATABASE_SIZE"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_PLATFORM_PREF"          0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_PREFERENCES$"           0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_PROVISION_COMPANY"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_PROVISION_SERICE_MOD"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_PURGED_SESSIONS$"       0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_QB_SAVED_COND"          0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_QB_SAVED_JOIN"          0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_QB_SAVED_QUERY"         0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_QB_SAVED_TABS"          0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_REGION_CHART_SER_ATTR"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_REGION_REPORT_FILTER"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_REPORT_LAYOUTS"         0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_REQUEST_VERIFICATIONS"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_REQUIRED_ROLES"         0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_RSCHEMA_EXCEPTIONS"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_SC_TRANS"               0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_SESSIONS$"              0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_SHARED_QRY_SQL_STMTS"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_SHARED_QUERIES"         0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_SHARED_WEB_SERVICES"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_SHORTCUT_USAGE_MAP"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_STEP_BRANCH_ARGS"       0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_SW_BINDS"               0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_SW_DETAIL_RESULTS"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_SW_RESULTS"             0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_SW_SQL_CMDS"            0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_SW_STMTS"               0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_TEMPLATES$"             0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_TEMPLATE_PREFERENCES"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_TEMPLATE_THEMES$"       0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_TRANSLATABLE_TEXT$"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_TREE_STATE"             0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_UPG_COL_NAME_CHANGES"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_USER_ACCESS_LOG1$"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_USER_ACCESS_LOG2$"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_VERSION$"               0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_WEB_PAGES"              0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_WEB_PG_LIST_ENTRIES"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_WEB_PG_REGIONS"         0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_WORKSHEET_CATEGORIES"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_WORKSHEET_COL_GROUPS"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_WORKSHEET_COMPUTATION"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_WORKSHEET_CONDITIONS"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_WORKSHEET_DOCS"         0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_WORKSHEET_GEOCACHE"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_WORKSHEET_HISTORY"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_WORKSHEET_LINKS"        0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_WORKSHEET_LOVS"         0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_WORKSHEET_LOV_ENTRIES"      0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_WORKSHEET_PRIVS"        0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_WORKSHEET_ROWS"         0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_WORKSHEET_STICK"        0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_WS_OPERATIONS"          0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_WS_PARAMETERS"          0 KB       0 rows
. . exported "APEX_030200"."WWV_FLOW_WS_PROCESS_PARMS_MAP"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACCESS"                  0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACC_COLUMNS"             0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACC_FORMS"               0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACC_FORMS_CONTROLS"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACC_FORMS_MODULES"       0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACC_FORMS_PERM"          0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACC_GROUPS"              0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACC_GRPSMEMBERS"         0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACC_INDEXES"             0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACC_INDEXES_COLS"        0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACC_MODULES"             0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACC_MODULES_PERM"        0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACC_PAGES"               0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACC_QUERIES"             0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACC_RELATIONS"           0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACC_RELATION_COLS"       0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACC_REPORTS"             0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACC_RPTS_CONTROLS"       0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACC_RPTS_MODULES"        0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACC_RPT_PERMS"           0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACC_TABLES"              0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACC_TAB_PERM"            0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_ACC_USERS"               0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FORMS"                   0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_ALERTS"              0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_ATTACHEDLIBRARY"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_BLK_DSA"             0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_BLK_DSC"             0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_BLK_ITEMS"           0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_BLK_ITEM_LIE"        0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_BLK_ITEM_RADIO"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_BLK_ITEM_TRIGGERS"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_BLK_RELATIONS"       0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_BLK_TRIGGERS"        0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_BLOCKS"              0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_CANVAS"              0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_CNVG_COMPOUNDTEXT"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_CNVS_GRAPHICS"       0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_CNVS_TABPAGE"        0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_COORDINATES"         0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_CPDTXT_TEXTSEGMENT"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_EDITOR"              0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_FMB_MENU"            0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_FMB_MENUITEM_ROLE"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_FMB_MENU_MENUITEM"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_FORMMODULES"         0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_LOV"                 0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_LOVCOLUMNMAPPING"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_MENU"                0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_MENUITEM_ROLE"       0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_MENUS"               0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_MENUSMODULEROLES"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_MENUS_MENUMODULES"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_MENUS_MODULES"       0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_MENUS_PROGRAMUNIT"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_MENU_MENUITEM"       0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_MODULEPARAMETER"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_MODULES"             0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_OBJECTGROUP"         0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_OBJECTGROUPCHILD"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_PROGRAMUNIT"         0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_PROPERTYCLASS"       0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_RECORDGROUPCOLUMN"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_RECORDGROUPS"        0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_REPORT"              0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_REV_APEX_APP"        0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_REV_BLK_ITEMS"       0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_REV_BLOCKS"          0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_REV_FORMMODULES"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_REV_LOV"             0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_REV_LOVCOLMAPS"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_TRIGGERS"            0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_VISUALATTRIBUTES"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_FRM_WINDOWS"             0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_GENERATED_APPLICATIONS"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB"                     0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_BLK_DATASOURCECOL"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_BLK_ITEM"            0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_BLK_ITEM_LIE"        0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_BLK_ITEM_TRIGGER"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_BLK_TRIGGER"         0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_BLOCK"               0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_CANVAS"              0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_CG_COMPOUNDTEXT"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_CG_CT_TEXTSEGMENT"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_CNVS_GRAPHICS"       0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_MODULES"             0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_OBJECTLIBRARY"       0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_OBJECTLIBRARYTAB"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_OLT_ALERT"           0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_OLT_BLK_ITEM"        0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_OLT_BLK_ITEM_TRIGR"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_OLT_BLOCK"           0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_OLT_CANVAS"          0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_OLT_CNVS_GRAPHICS"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_OLT_GRAPHICS"        0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_OLT_ITEM"            0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_OLT_MENU"            0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_OLT_MENU_MENUITEM"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_OLT_OBJECTGROUP"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_OLT_OB_OBJGRPCHILD"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_OLT_REPORT"          0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_OLT_TABPAGE"         0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_OLT_TABPG_GRAPHICS"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_OLT_VISUALATTRBUTE"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_OLT_WINDOW"          0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_PROGRAMUNIT"         0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_PROPERTYCLASS"       0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_T_TP_GGGGG_CPDTXT"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_T_TP_GGGGG_CT_TXST"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_T_TP_GGGG_CPDTXT"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_T_TP_GGGG_CT_TXSGT"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_T_TP_GGGG_GRAPHICS"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_T_TP_GGG_CPDTXT"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_T_TP_GGG_CT_TXTSGT"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_T_TP_GGG_GRAPHICS"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_T_TP_GG_CPDTXT"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_T_TP_GG_CT_TXTSGT"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_T_TP_GG_GRAPHICS"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_T_TP_G_GRAPHICS"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_VISUALATTRIBUTE"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_OLB_WINDOW"              0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_PLSQL_LIBS"              0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_PROJECTS"                0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_PROJECT_COMPONENTS"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_PROJECT_TRIGGERS"        0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_REPORT"                  0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_REV_APEXAPP"             0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_REV_FORMS"               0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_REV_QUERIES"             0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_REV_REPORTS"             0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_REV_TABLES"              0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_RPTS"                    0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_RPT_DATA"                0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_RPT_DATASRC"             0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_RPT_DATASRC_GRP"         0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_RPT_DATASRC_SELECT"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_RPT_DATA_SUMMARY"        0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_RPT_GRP_DATAITEM"        0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_RPT_GRP_DATAITEM_DESC"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_RPT_GRP_DATAITEM_PRIV"      0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_RPT_GRP_FIELD"           0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_RPT_GRP_FILTER"          0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_RPT_GRP_FORMULA"         0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_RPT_GRP_ROWDELIM"        0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_RPT_GRP_SUMMARY"         0 KB       0 rows
. . exported "APEX_030200"."WWV_MIG_RPT_REPORTPRIVATE"       0 KB       0 rows
. . exported "FLOWS_FILES"."WWV_FLOW_FILE_OBJECTS$"          0 KB       0 rows
. . exported "IX"."AQ$_ORDERS_QUEUETABLE_G"                  0 KB       0 rows
. . exported "IX"."AQ$_ORDERS_QUEUETABLE_H"                  0 KB       0 rows
. . exported "IX"."AQ$_ORDERS_QUEUETABLE_I"                  0 KB       0 rows
. . exported "IX"."AQ$_ORDERS_QUEUETABLE_L"                  0 KB       0 rows
. . exported "IX"."AQ$_ORDERS_QUEUETABLE_T"                  0 KB       0 rows
. . exported "IX"."AQ$_STREAMS_QUEUE_TABLE_C"                0 KB       0 rows
. . exported "IX"."AQ$_STREAMS_QUEUE_TABLE_G"                0 KB       0 rows
. . exported "IX"."AQ$_STREAMS_QUEUE_TABLE_H"                0 KB       0 rows
. . exported "IX"."AQ$_STREAMS_QUEUE_TABLE_I"                0 KB       0 rows
. . exported "IX"."AQ$_STREAMS_QUEUE_TABLE_L"                0 KB       0 rows
. . exported "IX"."AQ$_STREAMS_QUEUE_TABLE_T"                0 KB       0 rows
. . exported "IX"."ORDERS_QUEUETABLE"                        0 KB       0 rows
. . exported "IX"."STREAMS_QUEUE_TABLE"                      0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_ANON_ATTRS_WRK"               0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_ANON_RULES_WRK"               0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_CT_ACTION_WRK"                0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_CT_DAREFS_WRK"                0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_CT_LOCATORPATHS_WRK"          0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_CT_MACRO_DEP_WRK"             0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_CT_MACRO_PAR_WRK"             0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_CT_PRED_OPRD_WRK"             0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_CT_PRED_PAR_WRK"              0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_CT_PRED_SET_WRK"              0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_CT_PRED_WRK"                  0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_CT_VLD_MSG"                   0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_DATA_MODEL_WRK"               0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_DICT_ATTRS_WRK"               0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_DOCS_WRK"                     0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_DOC_REFS_WRK"                 0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_MAPPED_PATHS"                 0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_MAPPED_PATHS_WRK"             0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_MAPPING_DOCS_WRK"             0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_PRV_ATTRS_WRK"                0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_RT_PREF_PARAMS_WRK"           0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_STD_ATTRS_WRK"                0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_STORED_TAGS"                  0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_STORED_TAGS_WRK"              0 KB       0 rows
. . exported "ORDDATA"."ORDDCM_UID_DEFS_WRK"                 0 KB       0 rows
. . exported "OUTLN"."OL$"                                   0 KB       0 rows
. . exported "OUTLN"."OL$HINTS"                              0 KB       0 rows
. . exported "OUTLN"."OL$NODES"                              0 KB       0 rows
. . exported "OWBSYS"."OWBRTPS"                              0 KB       0 rows
. . exported "SCOTT"."BONUS"                                 0 KB       0 rows
. . exported "SH"."COSTS":"COSTS_1995"                       0 KB       0 rows
. . exported "SH"."COSTS":"COSTS_1996"                       0 KB       0 rows
. . exported "SH"."COSTS":"COSTS_H1_1997"                    0 KB       0 rows
. . exported "SH"."COSTS":"COSTS_H2_1997"                    0 KB       0 rows
. . exported "SH"."COSTS":"COSTS_Q1_2002"                    0 KB       0 rows
. . exported "SH"."COSTS":"COSTS_Q1_2003"                    0 KB       0 rows
. . exported "SH"."COSTS":"COSTS_Q2_2002"                    0 KB       0 rows
. . exported "SH"."COSTS":"COSTS_Q2_2003"                    0 KB       0 rows
. . exported "SH"."COSTS":"COSTS_Q3_2002"                    0 KB       0 rows
. . exported "SH"."COSTS":"COSTS_Q3_2003"                    0 KB       0 rows
. . exported "SH"."COSTS":"COSTS_Q4_2002"                    0 KB       0 rows
. . exported "SH"."COSTS":"COSTS_Q4_2003"                    0 KB       0 rows
. . exported "SH"."DIMENSION_EXCEPTIONS"                     0 KB       0 rows
. . exported "SH"."SALES":"SALES_1995"                       0 KB       0 rows
. . exported "SH"."SALES":"SALES_1996"                       0 KB       0 rows
. . exported "SH"."SALES":"SALES_H1_1997"                    0 KB       0 rows
. . exported "SH"."SALES":"SALES_H2_1997"                    0 KB       0 rows
. . exported "SH"."SALES":"SALES_Q1_2002"                    0 KB       0 rows
. . exported "SH"."SALES":"SALES_Q1_2003"                    0 KB       0 rows
. . exported "SH"."SALES":"SALES_Q2_2002"                    0 KB       0 rows
. . exported "SH"."SALES":"SALES_Q2_2003"                    0 KB       0 rows
. . exported "SH"."SALES":"SALES_Q3_2002"                    0 KB       0 rows
. . exported "SH"."SALES":"SALES_Q3_2003"                    0 KB       0 rows
. . exported "SH"."SALES":"SALES_Q4_2002"                    0 KB       0 rows
. . exported "SH"."SALES":"SALES_Q4_2003"                    0 KB       0 rows
. . exported "SYSMAN"."AQ$_MGMT_LOADER_QTABLE_G"             0 KB       0 rows
. . exported "SYSMAN"."AQ$_MGMT_LOADER_QTABLE_H"             0 KB       0 rows
. . exported "SYSMAN"."AQ$_MGMT_LOADER_QTABLE_I"             0 KB       0 rows
. . exported "SYSMAN"."AQ$_MGMT_LOADER_QTABLE_L"             0 KB       0 rows
. . exported "SYSMAN"."AQ$_MGMT_LOADER_QTABLE_T"             0 KB       0 rows
. . exported "SYSMAN"."AQ$_MGMT_NOTIFY_QTABLE_G"             0 KB       0 rows
. . exported "SYSMAN"."AQ$_MGMT_NOTIFY_QTABLE_H"             0 KB       0 rows
. . exported "SYSMAN"."AQ$_MGMT_NOTIFY_QTABLE_I"             0 KB       0 rows
. . exported "SYSMAN"."AQ$_MGMT_NOTIFY_QTABLE_L"             0 KB       0 rows
. . exported "SYSMAN"."AQ$_MGMT_NOTIFY_QTABLE_T"             0 KB       0 rows
. . exported "SYSMAN"."DB_USER_PREFERENCES"                  0 KB       0 rows
. . exported "SYSMAN"."EMDW_TRACE_DATA"                      0 KB       0 rows
. . exported "SYSMAN"."EM_COMPARISON_SUMMARY"                0 KB       0 rows
. . exported "SYSMAN"."EM_IPW_INFO"                          0 KB       0 rows
. . exported "SYSMAN"."EM_PAGE_CUSTOMIZATIONS"               0 KB       0 rows
. . exported "SYSMAN"."EM_PAGE_CUSTOM_CONDITIONS"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_ADMIN_METRIC_THRESHOLDS"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_ANNOTATION"                      0 KB       0 rows
. . exported "SYSMAN"."MGMT_ARU_CREDENTIALS"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_AUDIT_CUSTOM_ATTRIBS"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_AUDIT_LOGS"                      0 KB       0 rows
. . exported "SYSMAN"."MGMT_AVAILABILITY_RBK"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_BACKUP_CONFIGURATION"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_BAM_DATA_HUBS"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_BAM_DATA_ISESSIONS"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_BAM_DATA_OSESSIONS"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_BAM_ISESSION_DATASOURCE"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_BAM_ISESSION_DIAG"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_BAM_ISESSION_KPIS"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_BAM_OSESSION_ALERTS"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_BAM_OSESSION_DIAG"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_BAM_OSESSION_METRICS"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_BAM_OSESSION_STATUS"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_BCN_AVAIL_DEF"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_BCN_AVAIL_JOB"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_BCN_AVAIL_LOG"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_BCN_BCNSTEP_PROPS"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_BCN_BCNTXN_PROPS"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_BCN_STEPGROUP_DEFN"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_BCN_STEPGROUP_STEPS"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_BCN_STEP_DEFN"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_BCN_STEP_PROPS"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_BCN_TARGET"                      0 KB       0 rows
. . exported "SYSMAN"."MGMT_BCN_TXN_AUDIT"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_BCN_TXN_DEFN"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_BCN_TXN_PROPS"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_BLACKOUTS"                       0 KB       0 rows
. . exported "SYSMAN"."MGMT_BLACKOUT_FLAT_TARGETS"           0 KB       0 rows
. . exported "SYSMAN"."MGMT_BLACKOUT_HISTORY"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_BLACKOUT_SCHEDULE"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_BLACKOUT_STATE"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_BLACKOUT_TARGET_DETAILS"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_BLACKOUT_WINDOWS"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_BSLN_BASELINES"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_BSLN_DATASOURCES"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_BSLN_INTERVALS"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_BSLN_RAWDATA"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_BSLN_STATISTICS"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_BSLN_THRESHOLD_PARMS"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_BUG_ADVISORY"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_BUG_ADVISORY_BUG"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_BUG_ADV_HOME_PATCH"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_BUG_AVAILABLE_PATCH"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_BUG_FIX_APPLICABLE_COMP"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_BUG_FIX_APPLIC_COMP_LIST"        0 KB       0 rows
. . exported "SYSMAN"."MGMT_BUG_PATCH_CERTIFICATE"           0 KB       0 rows
. . exported "SYSMAN"."MGMT_BUG_PATCH_FIXES_BUG"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_BUG_PATCH_PLATFORM"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_CHANGE_AGENT_URL"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_BASELINES"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_BASELINE_CONS_GROUPS"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_BASELINE_DEPENDENCIES"        0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_BASELINE_DEPENDENTS"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_BASELINE_INIT_PARAMS"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_BASELINE_OBJECTS"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_BASELINE_OBJGRANTS"           0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_BASELINE_PROXYGRANTS"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_BASELINE_QUOTAGRANTS"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_BASELINE_ROLEGRANTS"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_BASELINE_SYSGRANTS"           0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_BASELINE_VERSIONS"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_COMPARISONS"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_COMPARISON_INIT_PRMS"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_COMPARISON_OBJECTS"           0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_COMPARISON_VERSIONS"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_SCHEMA_MAPS"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_SCOPESPECS"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_SCOPESPEC_NAMES"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_SYNCHRONIZATIONS"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_SYNCH_IMPACT_REPORTS"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_SYNCH_OBJECTS"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_SYNCH_SCRIPTS"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_CM_SYNCH_VERSIONS"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_COLLECTION_CREDENTIALS"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_COLLECTION_TASK_CONTEXT"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_COLLECTION_TEMPLATE_CREDS"       0 KB       0 rows
. . exported "SYSMAN"."MGMT_COLLECTION_WORKERS"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_COMP_RESULT_TO_JOB_MAP"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_COMP_SNAPSHOT_TO_STEP_MAP"       0 KB       0 rows
. . exported "SYSMAN"."MGMT_CONFIG_ACTIVITIES"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_CONTAINER_CREDENTIALS"           0 KB       0 rows
. . exported "SYSMAN"."MGMT_CPF_METRIC_SOURCE"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_CREDENTIALS"                     0 KB       0 rows
. . exported "SYSMAN"."MGMT_CSTMZ_CHARTS"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_CSTMZ_CHART_SELTARGETS"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_CSTMZ_CUSTOM_COLUMNS"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_CSTMZ_DEFAULT_CHART"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_CSTMZ_SUMMARY_CHART_DEF"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_CS_EVAL_SUMM_RQS"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_CS_EVAL_SUMM_RULE"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_CS_EVAL_SUMM_RULEFOLDER"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_CS_INCLUSION"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_CS_INCLUSION_PARAMETER"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_CS_PARAMETER"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_CS_PARAMETER_CHOICES"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_CS_REUSABLE_QUERY"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_CS_RQS_HIERARCHY"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_CS_RQS_INCLUSION"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_CS_RULE_FIX_LINK"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_CS_RULE_SIMPLE_TEST"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_CS_RULE_VIOL_CTX"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_CS_SCHEDULED_EVAL"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_DBNET_TNS_ADMINS"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_DB_INVOBJS_ECM"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_DB_LATEST_HDM_FINDINGS"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_DELTA_COMPARISON_DELTAS"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_DELTA_COMP_DELTA_DETAILS"        0 KB       0 rows
. . exported "SYSMAN"."MGMT_DELTA_COMP_KEY_COLS"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_DELTA_COMP_PROPERTIES"           0 KB       0 rows
. . exported "SYSMAN"."MGMT_DELTA_SUMMARY_ERRORS"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_DELTA_COMP_SUMMARIES"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_DELTA_SAVED_COMPARISON"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_DIROBJ_USERS_HOTLIST"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_DM_COLUMN_RULES"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_DM_INFCONS_COLUMNS"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_DM_JOB_EXECUTIONS"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_DM_SCOPESPECS"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_DM_SS_COLUMNS"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_DUPLICATE_TARGETS"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_E2E_DETAILS"                     0 KB       0 rows
. . exported "SYSMAN"."MGMT_E2E_DETAILS_1DAY"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_E2E_DETAILS_1HOUR"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_E2E_JDBC"                        0 KB       0 rows
. . exported "SYSMAN"."MGMT_E2E_JDBC_1DAY"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_E2E_JDBC_1HOUR"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_E2E_SQL"                         0 KB       0 rows
. . exported "SYSMAN"."MGMT_E2E_SQL_1DAY"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_E2E_SQL_1HOUR"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_E2E_SQL_CONN"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_E2E_SQL_STMT"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_E2E_SUMMARY"                     0 KB       0 rows
. . exported "SYSMAN"."MGMT_E2E_SUMMARY_1DAY"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_E2E_SUMMARY_1HOUR"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_CLUSTER_NODE_INFO"           0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_CSA"                         0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_CSA_APPID_TARGET_MAP"        0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_CSA_COOKIES"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_CSA_CUSTOM"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_CSA_FAILED"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_CSA_GENERAL_INFO"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_CSA_RULES"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_CSA_SNAPSHOT_INFO"           0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_HOSTPATCH_COMPL_HIST"        0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_HOSTPATCH_GROUPS"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_HOSTPATCH_GROUP_REPOS"       0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_HOSTPATCH_HOSTS"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_HOSTPATCH_HOST_COMPL"        0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_HOSTPATCH_REPOS"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_HOSTPATCH_REPOS_PKGS"        0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_HOST_CONFIGS_TO_DEL"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_HW"                          0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_HW_CPU"                      0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_HW_IOCARD"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_HW_NIC"                      0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_LOADED_FILES"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_OS"                          0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_OS_COMPONENT"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_OS_FILESYSTEM"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_OS_PROPERTY"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_OS_REGISTERED_SW"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_OS_REGISTERED_SW_COMP"       0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_PATCH_CACHE"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_SAVEDHOSTCONFIG"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_ULN_CHANNELS"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_ULN_CHANNEL_PKGS"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_ULN_CH_ADV"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_ULN_SS_CHANNELS"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_ECM_ULN_STAGE_SERVERS"           0 KB       0 rows
. . exported "SYSMAN"."MGMT_EMX_CELL_CD_CONFIG"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_EMX_CELL_C_CONFIG"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_EMX_CELL_GD_CONFIG"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_EMX_CELL_IORM_CONFIG"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_EMX_CELL_L_CONFIG"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_EMX_CELL_PD_CONFIG"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_ENTERPRISE_CREDENTIALS"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_FAILED_CONFIG_ACTIVITIES"        0 KB       0 rows
. . exported "SYSMAN"."MGMT_FEATURES_MAPPING"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_FEATURE_PATCHES"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_FLAT_ROLE_GRANTS"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_GENSVC_AVAIL_BEACONS"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_GENSVC_AVAIL_CONFIG"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_GENSVC_AVAIL_EVENTS"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_GENSVC_AVAIL_JOB"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_GENSVC_AVAIL_TESTS"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_GENSVC_JOBS_DETAILS"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_GENSVC_TEST_AVAIL"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_GENSVC_TEST_AVAIL_MARKER"        0 KB       0 rows
. . exported "SYSMAN"."MGMT_GENSVC_TEST_CUR_AVAIL"           0 KB       0 rows
. . exported "SYSMAN"."MGMT_GENSVC_TMPL_VARS"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_GENSVC_UPDBCN_JOB"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_GENSVC_UPDBCN_JOB_TESTS"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_GROUP_CHART"                     0 KB       0 rows
. . exported "SYSMAN"."MGMT_GROUP_CHART_SELTARGETS"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_GROUP_CUSTOM_COLUMNS"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_GROUP_SUMMARY_CHART_DEF"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_HA_BACKUP"                       0 KB       0 rows
. . exported "SYSMAN"."MGMT_HA_CLS_INTR_CONN"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_HA_DG_TARGET_SUMMARY"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_HA_FILES_ECM"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_HA_INIT_PARAMS_ECM"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_HA_RAC_INTR_CONN"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_HC_FS_MOUNT_DETAILS"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_HC_IOCARD_DETAILS"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_HOST_CREDENTIALS"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_HTTP_SESSION_OBJECTS"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_INDEX_SIZES"                     0 KB       0 rows
. . exported "SYSMAN"."MGMT_INV_COMPONENT_PATCH"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_INV_FILE"                        0 KB       0 rows
. . exported "SYSMAN"."MGMT_INV_PATCH"                       0 KB       0 rows
. . exported "SYSMAN"."MGMT_INV_PATCHED_FILE"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_INV_PATCHED_FILE_COMP"           0 KB       0 rows
. . exported "SYSMAN"."MGMT_INV_PATCHSET"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_INV_PATCH_FIXED_BUG"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_INV_VERSIONED_PATCH"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_IP_EMAIL_REPORT"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_IP_PURGE_POLICY"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_IP_REPORT_ELEM_IMAGE"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_IP_REPORT_ELEM_TARGETS"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_IP_STORED_REPORT"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_JOB_ASSOC_PARAMS"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_JOB_BLACKOUT_ASSOC"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_JOB_CREDENTIALS"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_JOB_EMD_STATUS_QUEUE"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_JOB_EXEC_CRED_INFO"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_JOB_EXEC_EVENT_PARAMS"           0 KB       0 rows
. . exported "SYSMAN"."MGMT_JOB_EXEC_LOCKS"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_JOB_EXT_TARGETS"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_JOB_FLAT_TARGETS"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_JOB_NOTIFY_STATES"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_JOB_PURGE_TARGETS"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_JOB_QUEUES"                      0 KB       0 rows
. . exported "SYSMAN"."MGMT_JOB_STEP_COMMAND_LOG"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_JOB_STEP_TARGETS"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_JOB_TARGET"                      0 KB       0 rows
. . exported "SYSMAN"."MGMT_JOB_TYPE_DISPLAY_PARAM"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_JOB_TYPE_PARAM_DROPDOWNS"        0 KB       0 rows
. . exported "SYSMAN"."MGMT_LICENSES"                        0 KB       0 rows
. . exported "SYSMAN"."MGMT_LICENSE_CONFIRMATION"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_LOADER_QTABLE"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_MANAGEMENT_PLUGINS"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_MASTER_AGENT"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_METRIC_COLLECTIONS_REP"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_METRIC_DEPENDENCY"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_METRIC_DEPENDENCY_DETAILS"       0 KB       0 rows
. . exported "SYSMAN"."MGMT_MNTR_SET_COPIES"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_MP_CONTRIBUTORS"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_MP_CONTRIBUTOR_FILE"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_MP_DEPLOYMENTS"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_MP_DEPLOYMENT_ERRORS"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_MP_FILES"                        0 KB       0 rows
. . exported "SYSMAN"."MGMT_MP_FILE_PROPS"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_MP_GROUPS"                       0 KB       0 rows
. . exported "SYSMAN"."MGMT_MP_GROUP_MEMBERS"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_MP_MECHANISMS"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_MP_NLS_SUBSTITUTIONS"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_MP_PROPS"                        0 KB       0 rows
. . exported "SYSMAN"."MGMT_NESTED_JOB_CRED_INFO"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_NET_EVENTS"                      0 KB       0 rows
. . exported "SYSMAN"."MGMT_NOTIFICATION_LOG"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_NOTIFY_DEVICES"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_NOTIFY_DEVICE_PARAMS"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_NOTIFY_DEV_SCHEDULES"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_NOTIFY_EMAIL_GATEWAY"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_NOTIFY_INPUT_QTABLE"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_NOTIFY_NOTIFYEES"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_NOTIFY_QTABLE"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_NOTIFY_REQUEUE"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_NOTIFY_SCHEDULES"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_OB_ADMIN_CLIENT_DB"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_OSM_DISK_GROUP_ECM"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_PAF_COMP_JOBTYPE_MAPPINGS"       0 KB       0 rows
. . exported "SYSMAN"."MGMT_PAF_ENCRYPTED_STRINGS"           0 KB       0 rows
. . exported "SYSMAN"."MGMT_PAF_INSTANCES"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_PAF_MSG_QTABLE_1"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_PAF_MSG_QTABLE_2"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_PAF_NOTIFICATION_LOG"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_PAF_OMS_STATUS"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_PAF_PAR_FILES"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_PAF_STATES"                      0 KB       0 rows
. . exported "SYSMAN"."MGMT_PDP_HOST_SETTING"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_PDP_SETTINGS"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_PDP_SETTING_VALUES"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_PLANPROBLEM_FACTORS"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_PROV_ASN_DEPENDENCIES"           0 KB       0 rows
. . exported "SYSMAN"."MGMT_PROV_ASN_TARGETS"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_PROV_ASSIGNMENT"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_PROV_BOOTSERVER"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_PROV_CLUSTER"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_PROV_CLUSTER_NODES"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_PROV_DEFAULT_IMAGE"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_PROV_HISTORY"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_PROV_IP_RANGE"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_PROV_IP_RESERVED"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_PROV_NET_CONFIG"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_PROV_OPERATION"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_PROV_RPM_REP"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_PROV_STAGED_COMPS"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_PROV_STAGING_DIRS"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_PROV_SUITE_INSTANCE"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_PROV_SUITE_INST_MEMBERS"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_RAC_SERVICES"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_RCA_EVENT"                       0 KB       0 rows
. . exported "SYSMAN"."MGMT_RCA_EVENT_ASSOC"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_RCA_METRIC_PROPS"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_RCA_METRIC_TEST"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_RCA_RECOVERY"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_RCA_RUN"                         0 KB       0 rows
. . exported "SYSMAN"."MGMT_RCA_SUMMARY"                     0 KB       0 rows
. . exported "SYSMAN"."MGMT_RCA_TARGET_PROPS"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_RCA_TEST_RESULT"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_RCA_TRACE"                       0 KB       0 rows
. . exported "SYSMAN"."MGMT_RCVCAT_CONFIG"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_RCVCAT_REPOS"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_REPOS_TIME_COEFFICIENT"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_COOKIE_DATA"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_DOMAIN_1DAY"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_DOMAIN_1HOUR"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_DOMAIN_BOOTSTRAP"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_DOMAIN_DIST_1DAY"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_DOMAIN_DIST_1HOUR"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_DOMAIN_DIST_BOOTSTRAP"        0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_INCOMPLETE_LOADS"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_INCOMPLETE_LOADS_1DAY"        0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_INCOMPLETE_LOADS_1HOUR"       0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_IP_1DAY"                      0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_IP_1HOUR"                     0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_IP_BOOTSTRAP"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_IP_DIST_1DAY"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_IP_DIST_1HOUR"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_IP_DIST_BOOTSTRAP"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_METRICS_RAW"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_PR_MAPPING"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_PR_MAPPING_1DAY"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_PR_MAPPING_1HOUR"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_REGIONS"                      0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_REGION_ENTRIES"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_REGION_MAPPING"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_TARGET_PROPERTIES"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_URLS"                         0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_URL_1DAY"                     0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_URL_1HOUR"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_URL_BOOTSTRAP"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_URL_DIST_1DAY"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_URL_DIST_1HOUR"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_RT_URL_DIST_BOOTSTRAP"           0 KB       0 rows
. . exported "SYSMAN"."MGMT_SEVERITY_RBK"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_SL_METRICS"                      0 KB       0 rows
. . exported "SYSMAN"."MGMT_SL_METRICS_HISTORY"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_SL_RULES"                        0 KB       0 rows
. . exported "SYSMAN"."MGMT_SL_RULES_HISTORY"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_SQLPROBLEM_FACTORS"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_SQL_BIND_VARS"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_SQL_EVALUATION"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_SQL_METRIC_HELPER"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_SQL_PLAN"                        0 KB       0 rows
. . exported "SYSMAN"."MGMT_SQL_REUSE"                       0 KB       0 rows
. . exported "SYSMAN"."MGMT_SQL_SUMMARY"                     0 KB       0 rows
. . exported "SYSMAN"."MGMT_SWLIB_DATA_DIRECTORIES"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_SWLIB_ENTITIES"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_SWLIB_ENTITY_DATA"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_SWLIB_ENTITY_DOCUMENTS"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_SWLIB_ENTITY_PARAMETERS"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_SWLIB_ENTITY_PLATFORMS"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_SWLIB_ENTITY_REFERENCES"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_SWLIB_ENTITY_REVISIONS"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_SWLIB_MATURITY_STATUS"           0 KB       0 rows
. . exported "SYSMAN"."MGMT_SWLIB_REVISION_PARAMETERS"       0 KB       0 rows
. . exported "SYSMAN"."MGMT_SYSTEM_CHANGES"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_TABLE_SIZES"                     0 KB       0 rows
. . exported "SYSMAN"."MGMT_TARGETS_DELETE"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_TARGET_AGENT_ASSOC"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_TARGET_ASSOC"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_TARGET_ASSOC_INSTANCE"           0 KB       0 rows
. . exported "SYSMAN"."MGMT_TARGET_ASSOC_PROP"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_TARGET_ASSOC_PROP_DEFS"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_TARGET_BASELINES"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_TARGET_BASELINES_DATA"           0 KB       0 rows
. . exported "SYSMAN"."MGMT_TARGET_DELETE_CALLBACKS"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_TARGET_PENDING_ASSOCS"           0 KB       0 rows
. . exported "SYSMAN"."MGMT_TEMPLATES"                       0 KB       0 rows
. . exported "SYSMAN"."MGMT_TEMPLATE_COPIES"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_TEST"                            0 KB       0 rows
. . exported "SYSMAN"."MGMT_TEST_DEFAULT_PROMOTION"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_TEST_DEFAULT_THRESHOLDS"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_TEST_MCOLUMNS"                   0 KB       0 rows
. . exported "SYSMAN"."MGMT_TEST_METRICS"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_TEST_METRIC_PROPS"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_TEST_PROP"                       0 KB       0 rows
. . exported "SYSMAN"."MGMT_TEST_PROP_CHOICES"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_TEST_PROP_LEVEL"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_TEST_PROP_QUALIFIERS"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_TEST_PROP_UIGROUP"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_TEST_QUALIFIERS"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_TEST_TARGET_MAP"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_TEXTINDEX"                       0 KB       0 rows
. . exported "SYSMAN"."MGMT_TEXTINDEX_LOGS_INFO"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_TEXT_INDEX_STATS"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_TOPO_PAGE_BG_IMAGE"              0 KB       0 rows
. . exported "SYSMAN"."MGMT_TOPO_PAGE_OBJ_POS"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_TOPO_PAGE_PREF"                  0 KB       0 rows
. . exported "SYSMAN"."MGMT_UPDATE_COLL_CREDS_DATA"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_UPDATE_CREDENTIALS_DATA"         0 KB       0 rows
. . exported "SYSMAN"."MGMT_UPDATE_OPERATIONS"               0 KB       0 rows
. . exported "SYSMAN"."MGMT_UPDATE_OPERATIONS_DATA"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_UPDATE_OPERATIONS_DETAILS"       0 KB       0 rows
. . exported "SYSMAN"."MGMT_UPDATE_PDP_DATA"                 0 KB       0 rows
. . exported "SYSMAN"."MGMT_UPDATE_PDP_DATA_COPY"            0 KB       0 rows
. . exported "SYSMAN"."MGMT_UPDATE_PDP_DATA_MAP"             0 KB       0 rows
. . exported "SYSMAN"."MGMT_UPDATE_PROPERTIES_DATA"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_UPDATE_TEMPLATE_DATA_MAP"        0 KB       0 rows
. . exported "SYSMAN"."MGMT_UPDATE_THRESHOLDS_DATA"          0 KB       0 rows
. . exported "SYSMAN"."MGMT_URL_CACHE"                       0 KB       0 rows
. . exported "SYSMAN"."MGMT_URL_PROXY"                       0 KB       0 rows
. . exported "SYSMAN"."MGMT_USER_JOBS"                       0 KB       0 rows
. . exported "SYSMAN"."MGMT_USER_PREFERENCES"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_USER_REPORT_DEFS"                0 KB       0 rows
. . exported "SYSMAN"."MGMT_USER_SESSION"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_USER_TARGETS"                    0 KB       0 rows
. . exported "SYSMAN"."MGMT_USER_TEMPLATES"                  0 KB       0 rows
. . exported "SYSTEM"."DEF$_AQCALL"                          0 KB       0 rows
. . exported "SYSTEM"."DEF$_AQERROR"                         0 KB       0 rows
. . exported "SYSTEM"."DEF$_CALLDEST"                        0 KB       0 rows
. . exported "SYSTEM"."DEF$_DEFAULTDEST"                     0 KB       0 rows
. . exported "SYSTEM"."DEF$_DESTINATION"                     0 KB       0 rows
. . exported "SYSTEM"."DEF$_ERROR"                           0 KB       0 rows
. . exported "SYSTEM"."DEF$_LOB"                             0 KB       0 rows
. . exported "SYSTEM"."DEF$_ORIGIN"                          0 KB       0 rows
. . exported "SYSTEM"."DEF$_PROPAGATOR"                      0 KB       0 rows
. . exported "SYSTEM"."DEF$_PUSHED_TRANSACTIONS"             0 KB       0 rows
. . exported "SYSTEM"."MVIEW$_ADV_INDEX"                     0 KB       0 rows
. . exported "SYSTEM"."MVIEW$_ADV_PARTITION"                 0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_AUDIT_COLUMN"                 0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_COLUMN_GROUP"                 0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_CONFLICT"                     0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_DDL"                          0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_EXCEPTIONS"                   0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_EXTENSION"                    0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_FLAVORS"                      0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_FLAVOR_OBJECTS"               0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_GENERATED"                    0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_GROUPED_COLUMN"               0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_INSTANTIATION_DDL"            0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_KEY_COLUMNS"                  0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_OBJECT_PARMS"                 0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_PARAMETER_COLUMN"             0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_PRIORITY"                     0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_PRIORITY_GROUP"               0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_REFRESH_TEMPLATES"            0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_REPCAT"                       0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_REPCATLOG"                    0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_REPCOLUMN"                    0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_REPGROUP_PRIVS"               0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_REPOBJECT"                    0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_REPPROP"                      0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_REPSCHEMA"                    0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_RESOLUTION"                   0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_RESOLUTION_STATISTICS"        0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_RESOL_STATS_CONTROL"          0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_RUNTIME_PARMS"                0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_SITES_NEW"                    0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_SITE_OBJECTS"                 0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_SNAPGROUP"                    0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_TEMPLATE_OBJECTS"             0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_TEMPLATE_PARMS"               0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_TEMPLATE_REFGROUPS"           0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_TEMPLATE_SITES"               0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_TEMPLATE_TARGETS"             0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_USER_AUTHORIZATIONS"          0 KB       0 rows
. . exported "SYSTEM"."REPCAT$_USER_PARM_VALUES"             0 KB       0 rows
. . exported "SYSTEM"."SQLPLUS_PRODUCT_PROFILE"              0 KB       0 rows
Master table "SYSTEM"."SYS_EXPORT_FULL_01" successfully loaded/unloaded
******************************************************************************
Dump file set for SYSTEM.SYS_EXPORT_FULL_01 is:
  /u01/backup/oracle11g.dmp
Job "SYSTEM"."SYS_EXPORT_FULL_01" completed with 1 error(s) at Fri Feb 26 14:02:01 2016 elapsed 0 00:04:27


```


impdp
-----
###### 表导入
以test_dump,scott_table表导入为例子  
```
# 因为我目前只有一个oracle测试环境，所以首先我删除表test_dump,scott_table表，便于我导入相关的表
SQL> drop table test_dump;
Table dropped
SQL> drop table scott_table;
Table dropped

# tables指定需要导入的表
# directory存放dumpfile和logfile的地方
# dumpfile需要导入的备份文件
# logfile导入日志文件
# 导入scott用户的test_dump,scott_table表
[oracle@wing ~]$ impdp scott/scott@WINGDB tables=scott_table,test_dump directory=TEST_DIR dumpfile=scott_dump.dmp logfile=scott_imp.log
Import: Release 11.2.0.4.0 - Production on Fri Feb 26 12:05:05 2016
Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.
Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
Master table "SCOTT"."SYS_IMPORT_TABLE_01" successfully loaded/unloaded
Starting "SCOTT"."SYS_IMPORT_TABLE_01":  scott/********@WINGDB tables=scott_table,test_dump directory=TEST_DIR dumpfile=scott_dump.dmp logfile=scott_imp.log 
Processing object type TABLE_EXPORT/TABLE/TABLE
Processing object type TABLE_EXPORT/TABLE/TABLE_DATA
. . imported "SCOTT"."SCOTT_TABLE"                       5.445 KB       2 rows
. . imported "SCOTT"."TEST_DUMP"                         5.085 KB      10 rows
Job "SCOTT"."SYS_IMPORT_TABLE_01" successfully completed at Fri Feb 26 12:05:08 2016 elapsed 0 00:00:02

# 进入directory发现增加了impdp日志文件
-rw-r--r--. 1 oracle oinstall    919 Feb 26 12:05 scott_imp.log

# 使用scott用户查看表已经导入成功
SQL> select * from test_dump;
                                     ID
---------------------------------------
                                      1
                                      2
                                      3
                                      4
                                      5
                                      6
                                      7
                                      8
                                      9
                                     10
10 rows selected

SQL> select * from scott_table;
                                     ID NAME
--------------------------------------- ----------------
                                      1 Oracle
                                      2 MySQL
# table_exists_action=append允许导入到已经存在的表中
# 没有使用table_exists_action=append导入到已经存在的表结构导入失败
[oracle@wing ~]$ impdp scott/scott@WINGDB tables=scott_table,test_dump directory=TEST_DIR dumpfile=scott_dump.dmp logfile=scott_imp.log
Import: Release 11.2.0.4.0 - Production on Fri Feb 26 13:12:45 2016
Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.
Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
Master table "SCOTT"."SYS_IMPORT_TABLE_01" successfully loaded/unloaded
Starting "SCOTT"."SYS_IMPORT_TABLE_01":  scott/********@WINGDB tables=scott_table,test_dump directory=TEST_DIR dumpfile=scott_dump.dmp logfile=scott_imp.log 
Processing object type TABLE_EXPORT/TABLE/TABLE
ORA-39151: Table "SCOTT"."TEST_DUMP" exists. All dependent metadata and data will be skipped due to table_exists_action of skip
ORA-39151: Table "SCOTT"."SCOTT_TABLE" exists. All dependent metadata and data will be skipped due to table_exists_action of skip
Processing object type TABLE_EXPORT/TABLE/TABLE_DATA
Job "SCOTT"."SYS_IMPORT_TABLE_01" completed with 2 error(s) at Fri Feb 26 13:12:46 2016 elapsed 0 00:00:01
# 使用table_exists_action=append导入到已经存在的表结构导入成功
[oracle@wing ~]$ impdp scott/scott@WINGDB tables=scott_table,test_dump directory=TEST_DIR dumpfile=scott_dump.dmp logfile=scott_imp.log table_exists_action=append
Import: Release 11.2.0.4.0 - Production on Fri Feb 26 13:12:35 2016
Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.
Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
Master table "SCOTT"."SYS_IMPORT_TABLE_01" successfully loaded/unloaded
Starting "SCOTT"."SYS_IMPORT_TABLE_01":  scott/********@WINGDB tables=scott_table,test_dump directory=TEST_DIR dumpfile=scott_dump.dmp logfile=scott_imp.log table_exists_action=append 
Processing object type TABLE_EXPORT/TABLE/TABLE
Table "SCOTT"."TEST_DUMP" exists. Data will be appended to existing table but all dependent metadata will be skipped due to table_exists_action of append
Table "SCOTT"."SCOTT_TABLE" exists. Data will be appended to existing table but all dependent metadata will be skipped due to table_exists_action of append
Processing object type TABLE_EXPORT/TABLE/TABLE_DATA
. . imported "SCOTT"."SCOTT_TABLE"                       5.445 KB       2 rows
. . imported "SCOTT"."TEST_DUMP"                         5.085 KB      10 rows
Job "SCOTT"."SYS_IMPORT_TABLE_01" successfully completed at Fri Feb 26 13:12:39 2016 elapsed 0 00:00:03
```

###### schema导入
以scott用户的schema为例  
```
[oracle@wing ~]$ impdp scott/scott@WINGDB schemas=SCOTT directory=TEST_DIR dumpfile=SCOTT.dmp logfile=scott_imp.log

Import: Release 11.2.0.4.0 - Production on Fri Feb 26 13:39:33 2016

Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
Master table "SCOTT"."SYS_IMPORT_SCHEMA_01" successfully loaded/unloaded
Starting "SCOTT"."SYS_IMPORT_SCHEMA_01":  scott/********@WINGDB schemas=SCOTT directory=TEST_DIR dumpfile=SCOTT.dmp logfile=scott_imp.log 
Processing object type SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
Processing object type SCHEMA_EXPORT/TABLE/TABLE
Processing object type SCHEMA_EXPORT/TABLE/TABLE_DATA
. . imported "SCOTT"."DEPT"                              5.929 KB       4 rows
. . imported "SCOTT"."EMP"                               8.562 KB      14 rows
. . imported "SCOTT"."SALGRADE"                          5.859 KB       5 rows
. . imported "SCOTT"."SCOTT_TABLE"                       5.468 KB       4 rows
. . imported "SCOTT"."TEST"                              5.062 KB       5 rows
. . imported "SCOTT"."TEST_DUMP"                         5.156 KB      20 rows
. . imported "SCOTT"."BONUS"                                 0 KB       0 rows
Processing object type SCHEMA_EXPORT/TABLE/INDEX/INDEX
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
Processing object type SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/INDEX_STATISTICS
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT
Processing object type SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
Job "SCOTT"."SYS_IMPORT_SCHEMA_01" successfully completed at Fri Feb 26 13:39:37 2016 elapsed 0 00:00:04

# 以scott用户验证schema导入
# 首先由于我只有一个oracle测试环境，所以需要先删除我的schema下的对象
SQL> select * from user_tables;
TABLE_NAME                     TABLESPACE_NAME                CLUSTER_NAME                   IOT_NAME                       STATUS     PCT_FREE   PCT_USED  INI_TRANS  MAX_TRANS INITIAL_EXTENT NEXT_EXTENT MIN_EXTENTS MAX_EXTENTS PCT_INCREASE  FREELISTS FREELIST_GROUPS LOGGING BACKED_UP   NUM_ROWS     BLOCKS EMPTY_BLOCKS  AVG_SPACE  CHAIN_CNT AVG_ROW_LEN AVG_SPACE_FREELIST_BLOCKS NUM_FREELIST_BLOCKS DEGREE                                   INSTANCES                                CACHE                TABLE_LOCK SAMPLE_SIZE LAST_ANALYZED PARTITIONED IOT_TYPE     TEMPORARY SECONDARY NESTED BUFFER_POOL FLASH_CACHE CELL_FLASH_CACHE ROW_MOVEMENT GLOBAL_STATS USER_STATS DURATION        SKIP_CORRUPT MONITORING CLUSTER_OWNER                  DEPENDENCIES COMPRESSION COMPRESS_FOR DROPPED READ_ONLY SEGMENT_CREATED RESULT_CACHE
------------------------------ ------------------------------ ------------------------------ ------------------------------ -------- ---------- ---------- ---------- ---------- -------------- ----------- ----------- ----------- ------------ ---------- --------------- ------- --------- ---------- ---------- ------------ ---------- ---------- ----------- ------------------------- ------------------- ---------------------------------------- ---------------------------------------- -------------------- ---------- ----------- ------------- ----------- ------------ --------- --------- ------ ----------- ----------- ---------------- ------------ ------------ ---------- --------------- ------------ ---------- ------------------------------ ------------ ----------- ------------ ------- --------- --------------- ------------
DEPT                           USERS                                                                                        VALID            10                     1        255          65536     1048576           1  2147483645                                         YES     N                  4          5            0          0          0          20                         0                   0          1                                        1                                   N                ENABLED              4 2016/2/14 10: NO                       N         N         NO     DEFAULT     DEFAULT     DEFAULT          DISABLED     YES          NO                         DISABLED     YES                                       DISABLED     DISABLED                 NO      NO        YES             DEFAULT
EMP                            USERS                                                                                        VALID            10                     1        255          65536     1048576           1  2147483645                                         YES     N                 14          5            0          0          0          38                         0                   0          1                                        1                                   N                ENABLED             14 2016/2/14 10: NO                       N         N         NO     DEFAULT     DEFAULT     DEFAULT          DISABLED     YES          NO                         DISABLED     YES                                       DISABLED     DISABLED                 NO      NO        YES             DEFAULT
BONUS                          USERS                                                                                        VALID            10                     1        255                                                                                            YES     N                  0          0            0          0          0           0                         0                   0          1                                        1                                   N                ENABLED              0 2016/2/14 10: NO                       N         N         NO     DEFAULT     DEFAULT     DEFAULT          DISABLED     YES          NO                         DISABLED     YES                                       DISABLED     DISABLED                 NO      NO        NO              DEFAULT
SALGRADE                       USERS                                                                                        VALID            10                     1        255          65536     1048576           1  2147483645                                         YES     N                  5          5            0          0          0          10                         0                   0          1                                        1                                   N                ENABLED              5 2016/2/14 10: NO                       N         N         NO     DEFAULT     DEFAULT     DEFAULT          DISABLED     YES          NO                         DISABLED     YES                                       DISABLED     DISABLED                 NO      NO        YES             DEFAULT
TEST                           USERS                                                                                        VALID            10                     1        255          65536     1048576           1  2147483645                                         YES     N                                                                                                                                     1                                        1                                   N                ENABLED                              NO                       N         N         NO     DEFAULT     DEFAULT     DEFAULT          DISABLED     NO           NO                         DISABLED     YES                                       DISABLED     DISABLED                 NO      NO        YES             DEFAULT
TEST_DUMP                      USERS                                                                                        VALID            10                     1        255          65536     1048576           1  2147483645                                         YES     N                                                                                                                                     1                                        1                                   N                ENABLED                              NO                       N         N         NO     DEFAULT     DEFAULT     DEFAULT          DISABLED     NO           NO                         DISABLED     YES                                       DISABLED     DISABLED                 NO      NO        YES             DEFAULT
SCOTT_TABLE                    USERS                                                                                        VALID            10                     1        255          65536     1048576           1  2147483645                                         YES     N                                                                                                                                     1                                        1                                   N                ENABLED                              NO                       N         N         NO     DEFAULT     DEFAULT     DEFAULT          DISABLED     NO           NO                         DISABLED     YES                                       DISABLED     DISABLED                 NO      NO        YES             DEFAULT
7 rows selected


SQL> drop table test;
Table dropped

SQL> drop table test_dump;
Table dropped

SQL> drop table scott_table;
Table dropped

SQL> drop table emp;
Table dropped

SQL> drop table dept;
Table dropped

SQL> drop table salgrade;
Table dropped

SQL> drop table bonus;
Table dropped

SQL> select  table_name from user_tables;
TABLE_NAME
------------------------------

SQL> select  * from user_tables;
TABLE_NAME                     TABLESPACE_NAME                CLUSTER_NAME                   IOT_NAME                       STATUS     PCT_FREE   PCT_USED  INI_TRANS  MAX_TRANS INITIAL_EXTENT NEXT_EXTENT MIN_EXTENTS MAX_EXTENTS PCT_INCREASE  FREELISTS FREELIST_GROUPS LOGGING BACKED_UP   NUM_ROWS     BLOCKS EMPTY_BLOCKS  AVG_SPACE  CHAIN_CNT AVG_ROW_LEN AVG_SPACE_FREELIST_BLOCKS NUM_FREELIST_BLOCKS DEGREE                                   INSTANCES                                CACHE                TABLE_LOCK SAMPLE_SIZE LAST_ANALYZED PARTITIONED IOT_TYPE     TEMPORARY SECONDARY NESTED BUFFER_POOL FLASH_CACHE CELL_FLASH_CACHE ROW_MOVEMENT GLOBAL_STATS USER_STATS DURATION        SKIP_CORRUPT MONITORING CLUSTER_OWNER                  DEPENDENCIES COMPRESSION COMPRESS_FOR DROPPED READ_ONLY SEGMENT_CREATED RESULT_CACHE
------------------------------ ------------------------------ ------------------------------ ------------------------------ -------- ---------- ---------- ---------- ---------- -------------- ----------- ----------- ----------- ------------ ---------- --------------- ------- --------- ---------- ---------- ------------ ---------- ---------- ----------- ------------------------- ------------------- ---------------------------------------- ---------------------------------------- -------------------- ---------- ----------- ------------- ----------- ------------ --------- --------- ------ ----------- ----------- ---------------- ------------ ------------ ---------- --------------- ------------ ---------- ------------------------------ ------------ ----------- ------------ ------- --------- --------------- ------------

# impdp之后再查看是否有相应的对象
SQL> select  * from user_tables;
TABLE_NAME                     TABLESPACE_NAME                CLUSTER_NAME                   IOT_NAME                       STATUS     PCT_FREE   PCT_USED  INI_TRANS  MAX_TRANS INITIAL_EXTENT NEXT_EXTENT MIN_EXTENTS MAX_EXTENTS PCT_INCREASE  FREELISTS FREELIST_GROUPS LOGGING BACKED_UP   NUM_ROWS     BLOCKS EMPTY_BLOCKS  AVG_SPACE  CHAIN_CNT AVG_ROW_LEN AVG_SPACE_FREELIST_BLOCKS NUM_FREELIST_BLOCKS DEGREE                                   INSTANCES                                CACHE                TABLE_LOCK SAMPLE_SIZE LAST_ANALYZED PARTITIONED IOT_TYPE     TEMPORARY SECONDARY NESTED BUFFER_POOL FLASH_CACHE CELL_FLASH_CACHE ROW_MOVEMENT GLOBAL_STATS USER_STATS DURATION        SKIP_CORRUPT MONITORING CLUSTER_OWNER                  DEPENDENCIES COMPRESSION COMPRESS_FOR DROPPED READ_ONLY SEGMENT_CREATED RESULT_CACHE
------------------------------ ------------------------------ ------------------------------ ------------------------------ -------- ---------- ---------- ---------- ---------- -------------- ----------- ----------- ----------- ------------ ---------- --------------- ------- --------- ---------- ---------- ------------ ---------- ---------- ----------- ------------------------- ------------------- ---------------------------------------- ---------------------------------------- -------------------- ---------- ----------- ------------- ----------- ------------ --------- --------- ------ ----------- ----------- ---------------- ------------ ------------ ---------- --------------- ------------ ---------- ------------------------------ ------------ ----------- ------------ ------- --------- --------------- ------------
DEPT                           USERS                                                                                        VALID            10                     1        255          65536     1048576           1  2147483645                                         YES     N                  4          5            0          0          0          20                         0                   0          1                                        1                                   N                ENABLED              4 2016/2/14 10: NO                       N         N         NO     DEFAULT     DEFAULT     DEFAULT          DISABLED     YES          NO                         DISABLED     YES                                       DISABLED     DISABLED                 NO      NO        YES             DEFAULT
EMP                            USERS                                                                                        VALID            10                     1        255          65536     1048576           1  2147483645                                         YES     N                 14          5            0          0          0          38                         0                   0          1                                        1                                   N                ENABLED             14 2016/2/14 10: NO                       N         N         NO     DEFAULT     DEFAULT     DEFAULT          DISABLED     YES          NO                         DISABLED     YES                                       DISABLED     DISABLED                 NO      NO        YES             DEFAULT
BONUS                          USERS                                                                                        VALID            10                     1        255                                                                                            YES     N                  0          0            0          0          0           0                         0                   0          1                                        1                                   N                ENABLED              0 2016/2/14 10: NO                       N         N         NO     DEFAULT     DEFAULT     DEFAULT          DISABLED     YES          NO                         DISABLED     YES                                       DISABLED     DISABLED                 NO      NO        NO              DEFAULT
SALGRADE                       USERS                                                                                        VALID            10                     1        255          65536     1048576           1  2147483645                                         YES     N                  5          5            0          0          0          10                         0                   0          1                                        1                                   N                ENABLED              5 2016/2/14 10: NO                       N         N         NO     DEFAULT     DEFAULT     DEFAULT          DISABLED     YES          NO                         DISABLED     YES                                       DISABLED     DISABLED                 NO      NO        YES             DEFAULT
TEST                           USERS                                                                                        VALID            10                     1        255          65536     1048576           1  2147483645                                         YES     N                                                                                                                                     1                                        1                                   N                ENABLED                              NO                       N         N         NO     DEFAULT     DEFAULT     DEFAULT          DISABLED     NO           NO                         DISABLED     YES                                       DISABLED     DISABLED                 NO      NO        YES             DEFAULT
TEST_DUMP                      USERS                                                                                        VALID            10                     1        255          65536     1048576           1  2147483645                                         YES     N                                                                                                                                     1                                        1                                   N                ENABLED                              NO                       N         N         NO     DEFAULT     DEFAULT     DEFAULT          DISABLED     NO           NO                         DISABLED     YES                                       DISABLED     DISABLED                 NO      NO        YES             DEFAULT
SCOTT_TABLE                    USERS                                                                                        VALID            10                     1        255          65536     1048576           1  2147483645                                         YES     N                                                                                                                                     1                                        1                                   N                ENABLED                              NO                       N         N         NO     DEFAULT     DEFAULT     DEFAULT          DISABLED     NO           NO                         DISABLED     YES                                       DISABLED     DISABLED                 NO      NO        YES             DEFAULT
7 rows selected
```

###### database导入
导入整个database  
```
[oracle@wing ~]$ impdp system/Wing_database@WINGDB full=Y directory=TEST_DIR dumpfile=oracle11g.dmp logfile=oracle11g_imp.log
# 结果不再展示
```


include/exclude的使用
--------------------
include表示只备份/恢复该字段指定的对象  
exclude表示除了该字段指定的对象都备份/恢复  
对象为object type(具体待以后补充)
```
# include导出
[oracle@wing ~]$ expdp scott/Wing_database@WINGDB tables=test,scott_table include=TABLE_DATA directory=TEST_DIR dumpfile=scott.dmp logfile=scott_exp.log
Export: Release 11.2.0.4.0 - Production on Fri Feb 26 14:56:58 2016
Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.
Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
Starting "SCOTT"."SYS_EXPORT_TABLE_01":  scott/********@WINGDB tables=test,scott_table include=TABLE_DATA directory=TEST_DIR dumpfile=scott.dmp logfile=scott_exp.log 
Estimate in progress using BLOCKS method...
Processing object type TABLE_EXPORT/TABLE/TABLE_DATA
Total estimation using BLOCKS method: 128 KB
. . exported "SCOTT"."SCOTT_TABLE"                       5.468 KB       4 rows
. . exported "SCOTT"."TEST"                              5.062 KB       5 rows
Master table "SCOTT"."SYS_EXPORT_TABLE_01" successfully loaded/unloaded
******************************************************************************
Dump file set for SCOTT.SYS_EXPORT_TABLE_01 is:
  /u01/backup/scott.dmp
Job "SCOTT"."SYS_EXPORT_TABLE_01" successfully completed at Fri Feb 26 14:57:00 2016 elapsed 0 00:00:02

# include导入
[oracle@wing ~]$ impdp scott/Wing_database@WINGDB tables=test,scott_table include=TABLE_DATA directory=TEST_DIR dumpfile=scott.dmp logfile=scott_imp.log
Import: Release 11.2.0.4.0 - Production on Fri Feb 26 15:00:44 2016
Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.
Connected to: Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
Master table "SCOTT"."SYS_IMPORT_TABLE_01" successfully loaded/unloaded
Starting "SCOTT"."SYS_IMPORT_TABLE_01":  scott/********@WINGDB tables=test,scott_table include=TABLE_DATA directory=TEST_DIR dumpfile=scott.dmp logfile=scott_imp.log 
Processing object type TABLE_EXPORT/TABLE/TABLE_DATA
. . imported "SCOTT"."SCOTT_TABLE"                       5.468 KB       4 rows
. . imported "SCOTT"."TEST"                              5.062 KB       5 rows
Job "SCOTT"."SYS_IMPORT_TABLE_01" successfully completed at Fri Feb 26 15:00:45 2016 elapsed 0 00:00:01

# exclude导出
用法同include

# exclude导入
用法通include
```


expdp参数详解
------------
[oracle@wing ~]$ expdp help=Y  
Export: Release 11.2.0.4.0 - Production on Fri Feb 26 15:17:53 2016  
Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.  
The Data Pump export utility provides a mechanism for transferring data objects
between Oracle databases. The utility is invoked with the following command:  
   Example: expdp scott/tiger DIRECTORY=dmpdir DUMPFILE=scott.dmp  
You can control how Export runs by entering the 'expdp' command followed  
by various parameters. To specify parameters, you use keywords:  
   Format:  expdp KEYWORD=value or KEYWORD=(value1,value2,...,valueN)  
   Example: expdp scott/tiger DUMPFILE=scott.dmp DIRECTORY=dmpdir SCHEMAS=scott
or TABLES=(T1:P1,T1:P2), if T1 is partitioned table  
USERID must be the first parameter on the command line.  

**The available keywords and their descriptions follow. Default values are listed within square brackets.**  

- ATTACH  
  Attach to an existing job.  
  For example, ATTACH=job_name.  

- CLUSTER  
  Utilize cluster resources and distribute workers across the Oracle RAC.
  Valid keyword values are: [Y] and N.

- COMPRESSION  
  Reduce the size of a dump file.  
  Valid keyword values are: ALL, DATA_ONLY, [METADATA_ONLY] and NONE.  

- CONTENT  
  Specifies data to unload.  
  Valid keyword values are: [ALL], DATA_ONLY and METADATA_ONLY.  
  指定需要导出的内容  

- DATA_OPTIONS  
  Data layer option flags.  
  Valid keyword values are: XML_CLOBS.  

- DIRECTORY  
  Directory object to be used for dump and log files.  
  指定dumpfile和logfile的存放位置  

- DUMPFILE  
  Specify list of destination dump file names [expdat.dmp].  
  For example, DUMPFILE=scott1.dmp, scott2.dmp, dmpdir:scott3.dmp.  
  expdp中指定dumpfile的文件位置  

- ENCRYPTION  
  Encrypt part or all of a dump file.  
  Valid keyword values are: ALL, DATA_ONLY, ENCRYPTED_COLUMNS_ONLY, METADATA_ONLY and NONE.  
  加密部分/全部的备份文件  

- ENCRYPTION_ALGORITHM  
  Specify how encryption should be done.  
  Valid keyword values are: [AES128], AES192 and AES256.  
  指定加密算法  

- ENCRYPTION_MODE  
  Method of generating encryption key.  
  Valid keyword values are: DUAL, PASSWORD and [TRANSPARENT].  
  生成加密密钥的方法  

- ENCRYPTION_PASSWORD  
  Password key for creating encrypted data within a dump file.  
  加密密码  

- ESTIMATE  
  Calculate job estimates.  
  Valid keyword values are: [BLOCKS] and STATISTICS.  

- ESTIMATE_ONLY  
  Calculate job estimates without performing the export.  

- EXCLUDE  
  Exclude specific object types.  
  For example, EXCLUDE=SCHEMA:"='HR'".  
  指定备份中需要排除的对象  

- FILESIZE  
  Specify the size of each dump file in units of bytes.  
  指定每个备份文件的大小（单位为byte）  

- FLASHBACK_SCN  
  SCN used to reset session snapshot.  
  指定导出特定SCN时刻的表数据  

- FLASHBACK_TIME  
  Time used to find the closest corresponding SCN value.  
  指定导出特定时间点的表数据  

- FULL  
  Export entire database [N].  
  备份整个数据库  

- HELP  
  Display Help messages [N].  
  显示帮助信息  

- INCLUDE  
  Include specific object types.  
  For example, INCLUDE=TABLE_DATA.  
  指定需要备份的对象  

- JOB_NAME  
  Name of export job to create.  

- LOGFILE  
  Specify log file name [export.log].  
  备份的日志文件  

- NETWORK_LINK  
  Name of remote database link to the source system.  

- NOLOGFILE  
  - Do not write log file [N].  
    不写备份日志文件  

- PARALLEL  
  Change the number of active workers for current job.  
  指定并发线程数，注意一个并发对应一个dumpfile,所以指定的并发线程数应该与dumpfile数量相同，dumpfile可以使用通配符%U进行匹配  
  例如：
  expdp ananda/abc123 tables=CASES directory=DPDATA1 dumpfile=expCASES_%U.dmp parallel=4 job_name=Cases_Export  

- PARFILE  
  Specify parameter file name.  
  指定expdp参数文件，文件以.par结束  

- QUERY  
  Predicate clause used to export a subset of a table.  
  For example, QUERY=employees:"WHERE department_id > 10".  
  按SQL语句条件备份  

- REMAP_DATA  
  Specify a data conversion function.  
  For example, REMAP_DATA=EMP.EMPNO:REMAPPKG.EMPNO.  
  将源对象的数据备份到目标对象的数据中  

- REUSE_DUMPFILES  
  Overwrite destination dump file if it exists [N].  

- SAMPLE  
  Percentage of data to be exported.  
  要导出的数据的百分比。  

- SCHEMAS  
  List of schemas to export [login schema].  
  需要备份的schema的列表  

- SERVICE_NAME  
  Name of an active Service and associated resource group to constrain Oracle RAC resources.  

- SOURCE_EDITION  
  Edition to be used for extracting metadata.  

- STATUS  
  Frequency (secs) job status is to be monitored where
  the default [0] will show new status when available.

- TABLES  
  Identifies a list of tables to export.  
  For example, TABLES=HR.EMPLOYEES,SH.SALES:SALES_1995.  
  需要备份的table的列表  

- TABLESPACES  
  Identifies a list of tablespaces to export.  
  需要备份的tablespace列表  

- TRANSPORTABLE  
  Specify whether transportable method can be used.  
  Valid keyword values are: ALWAYS and [NEVER].  

- TRANSPORT_FULL_CHECK  
  Verify storage segments of all tables [N].  

- TRANSPORT_TABLESPACES  
  List of tablespaces from which metadata will be unloaded.  

- VERSION  
  Version of objects to export.  
  Valid keyword values are: [COMPATIBLE], LATEST or any valid database version.  

**The following commands are valid while in interactive mode.**  
Note: abbreviations are allowed.  

- ADD_FILE  
  Add dumpfile to dumpfile set.  

- CONTINUE_CLIENT  
  Return to logging mode. Job will be restarted if idle.  

- EXIT_CLIENT
  Quit client session and leave job running.  

- FILESIZE  
  Default filesize (bytes) for subsequent ADD_FILE commands.  

- HELP  
  Summarize interactive commands.  

- KILL_JOB  
  Detach and delete job.  

- PARALLEL  
  Change the number of active workers for current job.  

- REUSE_DUMPFILES  
  Overwrite destination dump file if it exists [N].  

- START_JOB  
  Start or resume current job.  
  Valid keyword values are: SKIP_CURRENT.  

- STATUS  
  Frequency (secs) job status is to be monitored where  
  the default [0] will show new status when available.  

- STOP_JOB  
  Orderly shutdown of job execution and exits the client.  
  Valid keyword values are: IMMEDIATE.  


impdp参数详解
------------
[oracle@wing backup]$ impdp -help  
Import: Release 11.2.0.4.0 - Production on Fri Feb 26 16:26:24 2016  
Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.  
The Data Pump Import utility provides a mechanism for transferring data objects
between Oracle databases. The utility is invoked with the following command:  
     Example: impdp scott/tiger DIRECTORY=dmpdir DUMPFILE=scott.dmp  
You can control how Import runs by entering the 'impdp' command followed
by various parameters. To specify parameters, you use keywords:  
     Format:  impdp KEYWORD=value or KEYWORD=(value1,value2,...,valueN)
     Example: impdp scott/tiger DIRECTORY=dmpdir DUMPFILE=scott.dmp  
USERID must be the first parameter on the command line.  

**The available keywords and their descriptions follow. Default values are listed within square brackets.**  
- ATTACH  
  Attach to an existing job.  
  For example, ATTACH=job_name.  

- CLUSTER  
  Utilize cluster resources and distribute workers across the Oracle RAC.  
  Valid keyword values are: [Y] and N.  

- CONTENT  
  Specifies data to load.  
  Valid keywords are: [ALL], DATA_ONLY and METADATA_ONLY.  
  指定需要导入的数据  

- DATA_OPTIONS  
  Data layer option flags.  
  Valid keywords are: SKIP_CONSTRAINT_ERRORS.  

- DIRECTORY  
  Directory object to be used for dump, log and SQL files.  
  指定dumpfile和logfile的存放位置  

- DUMPFILE  
  List of dump files to import from [expdat.dmp].  
  For example, DUMPFILE=scott1.dmp, scott2.dmp, dmpdir:scott3.dmp.  
  备份文件的存放位置  

- ENCRYPTION_PASSWORD  
  Password key for accessing encrypted data within a dump file.  
  Not valid for network import jobs.  

- ESTIMATE  
  Calculate job estimates.  
  Valid keywords are: [BLOCKS] and STATISTICS.  

- EXCLUDE  
  Exclude specific object types.  
  For example, EXCLUDE=SCHEMA:"='HR'".  

- FLASHBACK_SCN  
  SCN used to reset session snapshot.  
  指定重新设置特定SCN

- FLASHBACK_TIME  
  Time used to find the closest corresponding SCN value.  
  指定找出最近的特定时间点的SCN值  

- FULL  
  Import everything from source [Y].  
  恢复整个数据库

- HELP  
  Display help messages [N].  
  显示帮助信息  

- INCLUDE  
  Include specific object types.  
  For example, INCLUDE=TABLE_DATA.  
  恢复指定的object type  

- JOB_NAME  
  Name of import job to create.  

- LOGFILE  
  Log file name [import.log].  
  指定import日志文件  

- NETWORK_LINK  
  Name of remote database link to the source system.  

- NOLOGFILE  
  Do not write log file [N].  
  不写impdp的日志文件  

- PARALLEL  
  Change the number of active workers for current job.  
  指定并发线程数  

- PARFILE  
  Specify parameter file.  
  指定impdp


- PARTITION_OPTIONS  
  Specify how partitions should be transformed.  
  Valid keywords are: DEPARTITION, MERGE and [NONE].  

- QUERY  
  Predicate clause used to import a subset of a table.  
  For example, QUERY=employees:"WHERE department_id > 10".  
  按SQL条件恢复  

- REMAP_DATA  
  Specify a data conversion function.  
  For example, REMAP_DATA=EMP.EMPNO:REMAPPKG.EMPNO.  

- REMAP_DATAFILE  
  Redefine data file references in all DDL statements.  

- REMAP_SCHEMA  
  Objects from one schema are loaded into another schema.  
  将源schema导入到目标schema  

- REMAP_TABLE  
  Table names are remapped to another table.  
  For example, REMAP_TABLE=HR.EMPLOYEES:EMPS.  
  将源表导入到目标表  

- REMAP_TABLESPACE  
  Tablespace objects are remapped to another tablespace.  
  将源表空间对象导入到目标表空间中  

- REUSE_DATAFILES  
  Tablespace will be initialized if it already exists [N].  

- SCHEMAS  
  List of schemas to import.
  指定恢复schema列表  

- SERVICE_NAME  
  Name of an active Service and associated resource group to constrain Oracle RAC resources.

- SKIP_UNUSABLE_INDEXES  
  Skip indexes that were set to the Index Unusable state.
  跳过不可用状态的索引  

- SOURCE_EDITION  
  Edition to be used for extracting metadata.  

- SQLFILE
  Write all the SQL DDL to a specified file.  
  将所有的DDL写到一个指定的文件中  

- STATUS  
  Frequency (secs) job status is to be monitored where  
  the default [0] will show new status when available.  

- STREAMS_CONFIGURATION  
  Enable the loading of Streams metadata  

- TABLE_EXISTS_ACTION  
  Action to take if imported object already exists.  
  Valid keywords are: APPEND, REPLACE, [SKIP] and TRUNCATE.  
  如果恢复数据时，对应的表存在，那么impdp才去的措施，append指将数据追加到已经存在的表中，replace指替换存在的表，skip指跳过已存在表的数据恢复，truncate指将已存在表的数据清空  

- TABLES  
  Identifies a list of tables to import.  
  For example, TABLES=HR.EMPLOYEES,SH.SALES:SALES_1995.  
  指定恢复table列表  

- TABLESPACES  
  Identifies a list of tablespaces to import.  
  指定恢复的tablespace列表  

- TARGET_EDITION  
  Edition to be used for loading metadata.  

- TRANSFORM  
  Metadata transform to apply to applicable objects.  
  Valid keywords are: OID, PCTSPACE, SEGMENT_ATTRIBUTES and STORAGE.  

- TRANSPORTABLE  
  Options for choosing transportable data movement.  
  Valid keywords are: ALWAYS and [NEVER].  
  Only valid in NETWORK_LINK mode import operations.  

- TRANSPORT_DATAFILES  
  List of data files to be imported by transportable mode.  

- TRANSPORT_FULL_CHECK  
  Verify storage segments of all tables [N].  

- TRANSPORT_TABLESPACES  
  List of tablespaces from which metadata will be loaded.  
  Only valid in NETWORK_LINK mode import operations.  

- VERSION  
  Version of objects to import.  
  Valid keywords are: [COMPATIBLE], LATEST or any valid database version.  
  Only valid for NETWORK_LINK and SQLFILE.  

**The following commands are valid while in interactive mode.**
Note: abbreviations are allowed.

- CONTINUE_CLIENT  
  Return to logging mode. Job will be restarted if idle.  

- EXIT_CLIENT  
  Quit client session and leave job running.  

- HELP  
  Summarize interactive commands.  

- KILL_JOB  
  Detach and delete job.  

- PARALLEL  
  Change the number of active workers for current job.  

- START_JOB  
  Start or resume current job.  
  Valid keywords are: SKIP_CURRENT.  

- STATUS  
  Frequency (secs) job status is to be monitored where the default [0] will show new status when available.  

- STOP_JOB  
  Orderly shutdown of job execution and exits the client.  
  Valid keywords are: IMMEDIATE.  
