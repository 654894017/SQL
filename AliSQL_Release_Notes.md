# AliSQL 5.6 Release Notes

## Abstract

AliSQL is a MySQL branch originated from Alibaba Group. It is based on the MySQL official release and has many feature and performance enhancements. AliSQL has proven to be very stable and efficient in production environment. It can be used as a free, fully compatible, enhanced and open source drop-in replacement for MySQL.

AliSQL has been an open source project since August 2016. It is being actively developed by engineers from Alibaba Group. Moreover, it includes patches from Percona, WebScaleSQL, and MariaDB. AliSQL is a fruit of community effort. Everyone is welcomed to get involved.

## Functionality Added or Changed

### 1. SELECT FOR UPDATE WAIT

**Description:**

This syntax provides a way to set lock wait timeout for some statement.

**Parameters:**

NO

**Usage:**

```SQL
1. SELECT ... FOR UPDATE [WAIT [n]|NO_WAIT]
2. SELECT ... LOCK IN SHARE MODE [WAIT [N]|NO_WAIT]
3. LOCK TABLE ... [WAIT [n]|NO_WAIT]
```

Timeout error:
"ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction;"

### 2. THD memory usage monitor

**Description:**

This feature supply a way to monitor thd or query memory usage.

- Memory\_used display current thread memory allocated.
- Query\_memory\_used display current thd->query() memory allocated.

**Parameters:**

NO

**Usage:**

```SQL
1. SHOW STATUS LIKE '%memory_used%'
2. SHOW FULL PROCESSLIST;
```

### 3. DDL fast fail

**Description:**

The feature provides a way to set lock wait timeout for DDL statements.

**Parameters:**

NO

**Usage:**

```SQL
1. ALTER TABLE T [WAIT [n]|NO_WAIT] ADD f INT
2. DROP TABLE [WAIT [n]|NO_WAIT]
```

### 4. Support big column compress

**Description:**

If the column was defined as a compressed column, then the column data will be
compressed using zlib. Currently blob/text/varchar/varbinary are supported.

**Parameters:**

1. innodb_rds_column_compression_level

    System Variable Name | innodb_rds_column_compression_level
    ---------------------|-------------------------------------------------------------------------------------------------------------------------------
    Variable Scope       | global
    Dynamic Variable     | YES
    Permitted Values     | [0, 9]
    Default              | 6
    Descirbe             | Compression level used for compress-format column.  0 is no compression  ,1 is fastest, 9 is best compression and default is 6

2. innodb_rds_column_zip_threshold

   System Variable Name | innodb_rds_column_zip_threshold
   ---------------------|----------------------------------------------------------
   Variable Scope       | global
   Dynamic Variable     | YES
   Permitted Values     | [1, ULONG_MAX]
   Default              | 96
   Descirbe             | Compress the column if the data length exceeds this value

3. innodb_rds_column_zlib_strategy

   System Variable Name | innodb_rds_column_zlib_strategy
   ---------------------|--------------------------------
   Variable Scope       | global
   Dynamic Variable     | YES
   Permitted Values     | [0, 4]
   Default              | 0
   Descirbe             | control the zlib strategy

4. innodb_rds_column_zlib_wrap

   System Variable Name | innodb_rds_column_zlib_wrap
   ---------------------|-------------------------------------------
   Variable Scope       | global
   Dynamic Variable     | YES
   Permitted Values     | [OFF, ON]
   Default              | ON
   Descirbe             | control if the compressed data is wrapped

5. innodb_rds_column_zip_mem_use_heap

   System Variable Name | innodb_rds_column_zip_mem_use_heap
   ---------------------|------------------------------------------------------------------------------
   Variable Scope       | global
   Dynamic Variable     | YES
   Permitted Values     | [OFF, ON]
   Default              | OFF
   Descirbe             | alloc memory from prebuilt->compress_heap for zlib during compress/decompress

**Usage:**

```SQL
CREATE TABLE t(
  a VARCHAR(100) COLUMN_FORMAT COMPRESSED
)CHARACTER SET GBK ENGINE=INNODB;
```

### 5. Innodb_rseg table to display the rollback information.

**Description:**

This feature introduced one system table "information_schema.innodb_rseg" to
show the status of each of the rollback segments information.

**Parameters:**

NO

**Usage:**

```SQL
SELECT * FROM INFORMATION_SCHEMA.INNODB_RSEG
```

### 6. Thread running control

**Description:**

This feature is used to keep database safe when database has a high load.

We introduced two system variables to help limit the database concurrency. One
of the variables is `rds_threads_running_high_watermark`, which is used to control
how many running connections the database allowes. The other variable is
`rds_threads_running_ctl_mode`, which defines what type of query we would like to
block. It can be only SELECT query or all kinds queries(such as all DMLs etc.).
When running connections are beyond `rds_threads_running_high_watermark`, all of the new
queries(up to 'rds_threads_running_ctl_mode'), will be rejected, i.e. an error will be
returned to client.

**Parameters:**

1. rds_threads_running_high_watermark

   System Variable Name | rds_threads_running_high_watermark
   ---------------------|----------------------------------------------------------------------------------------------
   Variable Scope       | global
   Dynamic Variable     | YES
   Permitted Values     | [0, 50000]
   Default              | 0
   Descirbe             | When threads_running exceeds this limit, query that isn't in active transaction should quit.

2. rds_threads_running_ctl_mode

   System Variable Name | rds_threads_running_ctl_mode
   ---------------------|----------------------------------------------------------------------
   Variable Scope       | global
   Dynamic Variable     | YES
   Permitted Values     | [SELECTS, ALL, 0]
   Default              | 0
   Descirbe             | Control which statements will be affected by threads running control

**Usage:**

no

### 7. Kill idle transactions

**Description:**

Terminate idle transactions safely in server layer by setting up socket timeout parameter.

**Parameters:**

1. trx_idle_timeout

   System Variable Name | trx_idle_timeout
   ---------------------|-------------------------------------------------------------
   Variable Scope       | session/global
   Dynamic Variable     | YES
   Permitted Values     | [0, LONG_TIMEOUT]
   Default              | 0
   Descirbe             | The number of seconds the server waits for idle transaction

2. trx_readonly_idle_timeout

   System Variable Name | trx_readonly_idle_timeout
   ---------------------|----------------------------------------------------------------------
   Variable Scope       | session/global
   Dynamic Variable     | YES
   Permitted Values     | [0, LONG_TIMEOUT]
   Default              | 0
   Descirbe             | The number of seconds the server waits for readonly idle transaction

3. trx_changes_idle_timeout

   System Variable Name | trx_changes_idle_timeout
   ---------------------|--------------------------------------------------------------------------
   Variable Scope       | session/global
   Dynamic Variable     | YES
   Permitted Values     | [0, LONG_TIMEOUT]
   Default              | 0
   Descirbe             | The number of seconds the server waits for not-readonly idle transaction

**Usage:**

NO


### 8. table/index statistics

**Description:**

Table statistics provides direct hint information for business logic split and stress analysis.
Index statistics provides hint information to improve performance of range query, point query, or join;
as far as index evaluation concerned,

**Parameters:**

1. rds_tablestat

   System Variable Name | rds_tablestat
   ---------------------|-------------------------
   Variable Scope       | global
   Dynamic Variable     | YES
   Permitted Values     | [OFF, ON]
   Default              | OFF
   Descirbe             | Control TABLE_STATISTICS

2. rds_indexstat

   System Variable Name | rds_indexstat
   ---------------------|--------------------------
   Variable Scope       | global
   Dynamic Variable     | YES
   Permitted Values     | [OFF, ON]
   Default              | OFF
   Descirbe             | Control INDEX_STATISTICS

**Usage:**

```SQL
SELECT * FROM TABLE_STATISTICS;
SELECT * FROM INDEX_STATISTICS;
```

### 9. Throttle InnoDB IOPS for sql statement

**Description:**

Supply three status variables:
1. logical_read: Buffer pool read
2. physical_sync_read: Disk direct read
3. physical_async_read: Disk async read triggered by read-ahead

Provide a way to control IOPS for one sql through rds_sql_max_iops.

**Parameters:**

1. rds_sql_max_iops

   System Variable Name | rds_sql_max_iops
   ---------------------|--------------------------
   Variable Scope       | session
   Dynamic Variable     | YES
   Permitted Values     | [0, ULONG_MAX]
   Default              | 0
   Descirbe             | Control TABLE_STATISTICS

**Usage:**

```SQL
SET SESSION RDS_SQL_MAX_IOPS=100;
```


### 10. SQL filter

**Description:**

This feature limit queries of SQL which contains key words.
Only SELECT, UPDATE, DELETE can be limited.

**Parameters:**

1. rds_sql_max_iops

   System Variable Name | rds_sql_max_iops
   ---------------------|--------------------------
   Variable Scope       | session
   Dynamic Variable     | YES
   Permitted Values     | [0, ULONG_MAX]
   Default              | 0
   Descirbe             | Control TABLE_STATISTICS

**Usage:**

```SQL
Add a rule limit SELECT:

SET GLOBAL sql_select_filter = '+,{CONC},KEY1~KEY2~KEY3...';
keys seperated with '~'
example: SET GLOBAL sql_select_filter = '+,1,t1~t2~t3'
SQL contains 't1','t2','t3' can only have one conncetion at the same time
sql_update_filter and sql_delete_filter for UPDATE and DELETE

show rules:
SHOW SQL_FILTERS;
select * from information_schema.sql_filter_info;

result example:
TYPE    ITEM_ID CUR_CONC        MAX_CONC        KEY_NUM KEY_STR
SELECT  2       0       0       2       +,0,a=1~a=2
SELECT  1       0       1       2       +,1,a=1~a=2

delete rules:
SET GLOBAL sql_select_filter = '-,ITEM_ID1,ITEM_ID2,.....';

delete all rules:
SET GLOBAL reset_all_filter = 1;
```

### 11. Relax gtid limitation for some statements

**Description:**

When gtid_mode is on, MySQL has a limitation on cts(create table ... select)
and the statements mixed with transaction/non-transaction storage engines.
It doesn't allow these statements to be executed. Such a limitation brings
many inconveniences to users.

In order to remove that limitation, we implemented such a feature which can
make those statments run under rbr(row based replication) mode and gtid mode.

We introduced one global system variable 'rds_allow_unsafe_stmt_with_gtid' to
enable such a feature. When the feature is enabled, cts will be divided into
two individual transactions. The first is CREATE TABLE. The second is inserting
data into the table. For the second transaction, it will write into binlog
directly, which means not to wait for any transaction commit once the first
transaction is succesfull. The same logicfor the statements mixing
nontransactional storage engine with transactional ones.

**Parameters:**

1. rds_allow_unsafe_stmt_with_gtid

   System Variable Name | rds_allow_unsafe_stmt_with_gtid
   ---------------------|---------------------------------------------------------------------------------
   Variable Scope       | global
   Dynamic Variable     | YES
   Permitted Values     | [OFF, ON]
   Default              | OFF
   Descirbe             | Allow executing CREATE TABLE AS SELECT or mixed engine transactions if enabled.

**Usage:**

NO

## Performance improvement

### 1. Optimize log_write_up_to logic

**Description:**

Optimize the redo log write logic:

1. Reduce log_sys->mutex contention.
2. Remove unnecessary ut_memcpy when log_group_write_buf
3. Keep one flush_event since we have only one redo log group

**Parameters:**

NO

**Usage:**

NO

### 2. Split LOCK_grant lock

**Description:**

Privileges will be checked before every query executed. so the Lock_grant that
protected ACLs is a hot mutex.
We split the LOCK_grant into an array of rw_locks. Require a read_mode rw_lock through [thread_id % array_size]
when check privilege, and require all write_mode rw_locks when modify privilegs.

**Parameters:**

NO

**Usage:**

NO

### 3. Build jemalloc statically into mysqld

**Description:**

Jemalloc memory allocator is very efficient compared with glibc malloc or TCmalloc.
For the deploy convenience, we build jemalloc statically.

**Parameters:**

NO

**Usage:**

NO

### 4. Redo log group commit at server layer

**Description:**

When binlog is on, InnoDB redo log will write at InnoDB prepare stage,
and binlog will write at binlog commit stage.

Actually, InnoDB redo log write action could move to binlog commit stage only
if it completed before binlog. Then, redo log and binlog can use original
binary group commit logic to improve performance.

**Parameters:**

NO

**Usage:**

NO

### 5. Merge InnoDB AIO request

**Description:**

The InnoDB engine support native AIO and simulated AIO on linux platform.
Native AIO use io_submit that glibc supplied to request IO.
But InnoDB engine requested AIO one by one through io_submit when trigger read-ahead,
so it is a litte inefficiency.

We buffered the AIO requests, then io_submit all.
For example: when linear-ahead. we buffered next 64 pages io requests, at last io_submit all io requests.

**Parameters:**

NO

**Usage:**

NO

### 6. Split redo log buffer to rotate log write

**Description:**

Allocate two global redo log buffer to rotate log write.

1. Split the log buffer into the two buffer that has the same size with innodb_log_buffer_size.
   So that when log_write_up_to_lsn occured, the mtr commit can write private log into
   another global log buffer.
2. Split log_sys->mutex into two mutex.
   log_sys->mutex protect the log buffer.
   log_sys->w_mutex protect log file.

Those optimization can reduce the log write conflict and improve throughout.

**Parameters:**

NO

**Usage:**

NO

### 7. Partition Lock_done and Lock_cond

**Description:**

When group commit occured. There will be have serval THD groups for every stage.
then split m_lock_done and m_lock_cond for group commit, each group
commit leader signal their owner follower in every group commit stage.

**Parameters:**

NO

**Usage:**

NO

### 8. Buffer pool list scan optimization

**Description:**

Optimize buffer pool list scans and related batch processing code:

  Reduce excessive scanning of pages when doing flush list batches. The
fix is to introduce the concept of "Hazard Pointer", this reduces the
time complexity of the scan from O(n*n) to O(n).

**Parameters:**

NO

**Usage:**

NO


### 9. Provide adaptive algorithm for innodb concurrency tickets

**Description:**

To avoide too many concurrency threads, InnoDB use innodb_thread_concurrency to
control the number of operating system threads concurrently inside InnoDB.
Innodb_concurrency_tickets is a fixed number, and it is hard to
choose a proper value fit for both small transactions and large transactons.

This patch provides a method to adaptively adjust tickets assigned to
readonly SELECT SQL statement. For large SELECT query, it may acquire
tickets many times if the previous assigned tickets are exhausted.
To avoid starving smally quries, the tickets assiged to large query are
decreased exponentially according to the number of times it acquire
tickets, i.e

**Parameters:**

1. innodb_rds_adaptive_tickets_algo

   System Variable Name | innodb_rds_adaptive_tickets_algo                 
   ---------------------|--------------------------------------------------
   Variable Scope       | global                                           
   Dynamic Variable     | YES                                              
   Permitted Values     | [FALSE, TRUE]                                    
   Default              | FALSE                                            
   Descirbe             | Whether to enable the adaptive tickets algorithm 

2. innodb_rds_min_concurrency_tickets

   System Variable Name | innodb_rds_min_concurrency_tickets                                                   
   ---------------------|--------------------------------------------------------------------------------------
   Variable Scope       | global                                                                               
   Dynamic Variable     | YES                                                                                  
   Permitted Values     | [1, ULONG_MAX]                                                                       
   Default              | 50                                                                                   
   Descirbe             | The lower bound of the concurrency_tickets when rds_adaptive_tickets_algo is enabled 

**Usage:**

NO

### 10. Optimize InnoDB read view creation

**Description:**

Read_view_open_now() and read_cursor_view_create_for_mysql() do a scan
of the list of all open transactions (trx_sys->trx_list) with the
kernel_mutex locked. On high-concurrency workloads (i.e. sysbench
read-only with number of threads >= 512) the cost of this scan is
huge.

The patch is ported from Percona Server, and contains 2 optimizations
to reduce the cost during read view creating.

1. Pre-allocate a read view structure for every transaction
2. Maintain a trx_id array. While creating a read view, simply copy
   the array without traversing the transaction rw_list.

**Parameters:**

NO

**Usage:**

NO


### 11. Optimize check/grant of InnoDB table lock

**Description:**

Since most workload in production environment is normal DML/SELECT
(this should be very common on most OLTP workload).
So for InnoDB table lock:
  1. most time there's only LOCK_IX/LOCK_IS table lock
  2. LOCK_S/LOCK_X rarely happen

This patch adds some counters to identify if LOCK_S/LOCK_X is granted
and quickly exit function lock_table_other_has_incompatible().
This will improve the performance especially when all DML focused on
the same table.

**Parameters:**

NO

**Usage:**

NO


### 12. Partition btr_search_latch for AHI by index->id

**Description:**

Partition btr_search_latch latch for AHI to decrease the concurrency contention.
Variable innodb_adaptive_hash_index_parts was introduced to
control the partition number of the latch.

**Parameters:**

1. innodb_adaptive_hash_index_parts

   System Variable Name | innodb_adaptive_hash_index_parts             
   ---------------------|----------------------------------------------
   Variable Scope       | global                                       
   Dynamic Variable     | NO                                           
   Permitted Values     | [1, 512]                                     
   Default              | 8                                            
   Descirbe             | The partition number of the btr_search_latch 

**Usage:**

NO

### 13. Omit maintaining owned_gtid set if gtid is auto-generated

**Description:**

Before this patch, the life cycle for a GTID like the following:
Generate a GTID => Add the GTID into owned_gtid => Flush the GTID into binlog
=> Remove the GTID from owned_gtid. Owned_gtid can be treated as an intermediate
set to hold all of the GTIDs of current thread.

After this patch, the life cycle for such a GTID can be like this:
Generate a GTID => Add the GTID into binlog. Of course, if anything
wrong happens, we have to remove the GTID from the binlog. Before this
patch, we can remove the GTID from owned_gtid. Obviously, we shortened
the life cycle of GTID.

**Parameters:**

1. rds_gtid_precommit

   System Variable Name | rds_gtid_precommit                                                     
   ---------------------|------------------------------------------------------------------------
   Variable Scope       | global                                                                 
   Dynamic Variable     | YES                                                                    
   Permitted Values     | [OFF, ON]                                                              
   Default              | OFF                                                                    
   Descirbe             | Add gtid into gtid_executed before flushing binlog from cache to file. 

**Usage:**

NO


### 14. Optimize InnoDB read-only workload

**Description:**

These path introduces serveral improvement for InnoDB read-only workload:

1. Remove trx_sys_t::ro_trx_list, all transaction will start with
read only mode, and change to rw mode while encountering DML
statement. For more details, refer to mysql worklog WL#6047.

2. Read view object is cached for read-only statement if no rw
statement happens during two read only statements of the same
session.

3. Ignore acquiring lock_sys mutex contention while commiting
read only transaction.

**Parameters:**

NO

**Usage:**

NO
