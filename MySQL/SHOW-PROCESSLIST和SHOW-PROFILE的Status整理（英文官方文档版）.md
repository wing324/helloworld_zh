## MySQL之SHOW PROCESSLIST和SHOW PROFILE的Status整理（英文官方文档版）

MySQL官方文档中的status存在于好几个小节中，每个status的英文解释其实一般是可以看的懂的，主要是寻找相应的status有点困难，所以我将所有的status整理在一个页面，直接Ctrl+F就可以查找到相应的status啦。  


Thread Command
--------------
A thread can have any of the following Command values:  
##### Binlog Dump
This is a thread on a master server for sending binary log contents to a slave server.

##### Change user
The thread is executing a change-user operation.

##### Close stmt
The thread is closing a prepared statement.

##### Connect
A replication slave is connected to its master.

##### Connect Out
A replication slave is connecting to its master.

##### Create DB
The thread is executing a create-database operation.

##### Daemon
This thread is internal to the server, not a thread that services a client connection.

##### Debug
The thread is generating debugging information.

##### Delayed insert
The thread is a delayed-insert handler.

##### Drop DB
The thread is executing a drop-database operation.

##### Error

##### Execute
The thread is executing a prepared statement.

##### Fetch
The thread is fetching the results from executing a prepared statement.

##### Field List
The thread is retrieving information for table columns.

##### Init DB
The thread is selecting a default database.

##### Kill
The thread is killing another thread.

##### Long Data
The thread is retrieving long data in the result of executing a prepared statement.

##### Ping
The thread is handling a server-ping request.

##### Prepare
The thread is preparing a prepared statement.

##### Processlist
The thread is producing information about server threads.

##### Query
The thread is executing a statement.

##### Quit
The thread is terminating.

##### Refresh
The thread is flushing table, logs, or caches, or resetting status variable or replication server information.

##### Register Slave
The thread is registering a slave server.

##### Reset stmt
The thread is resetting a prepared statement.

##### Set option
The thread is setting or resetting a client statement-execution option.

##### Shutdown
The thread is shutting down the server.

##### Sleep
The thread is waiting for the client to send a new statement to it.

##### Statistics
The thread is producing server-status information.

##### Table Dump
The thread is sending table contents to a slave server.

##### Time
Unused.


General Thread
--------------
The following list describes thread State values that are associated with general query processing and not more specialized activities such as replication. Many of these are useful only for finding bugs in the server.  
##### After create
This occurs when the thread creates a table (including internal temporary tables), at the end of the function that creates the table. This state is used even if the table could not be created due to some error.

##### altering table
The server is in the process of executing an in-place ALTER TABLE.

##### Analyzing
The thread is calculating a MyISAM table key distributions (for example, for ANALYZE TABLE).

##### checking permissions
The thread is checking whether the server has the required privileges to execute the statement.

##### Checking table
The thread is performing a table check operation.

##### cleaning up
The thread has processed one command and is preparing to free memory and reset certain state variables.

##### closing tables
The thread is flushing the changed table data to disk and closing the used tables. This should be a fast operation. If not, verify that you do not have a full disk and that the disk is not in very heavy use.

##### committing alter table to storage engine
The server has finished an in-place ALTER TABLE and is committing the result.

##### converting HEAP to MyISAM
The thread is converting an internal temporary table from a MEMORY table to an on-disk MyISAM table.

##### copy to tmp table
The thread is processing an ALTER TABLE statement. This state occurs after the table with the new structure has been created but before rows are copied into it.

##### Copying to group table
If a statement has different ORDER BY and GROUP BY criteria, the rows are sorted by group and copied to a temporary table.

##### Copying to tmp table
The server is copying to a temporary table in memory.

##### Copying to tmp table on disk
The server is copying to a temporary table on disk. The temporary result set has become too large (see Section 8.4.4, “Internal Temporary Table Use in MySQL”). Consequently, the thread is changing the temporary table from in-memory to disk-based format to save memory.

##### Creating index
The thread is processing ALTER TABLE ... ENABLE KEYS for a MyISAM table.

##### Creating sort index
The thread is processing a SELECT that is resolved using an internal temporary table.

##### creating table
The thread is creating a table. This includes creation of temporary tables.

##### Creating tmp table
The thread is creating a temporary table in memory or on disk. If the table is created in memory but later is converted to an on-disk table, the state during that operation will be Copying to tmp table on disk.

##### deleting from main table
The server is executing the first part of a multiple-table delete. It is deleting only from the first table, and saving columns and offsets to be used for deleting from the other (reference) tables.

##### deleting from reference tables
The server is executing the second part of a multiple-table delete and deleting the matched rows from the other tables.

##### discard_or_import_tablespace
The thread is processing an ALTER TABLE ... DISCARD TABLESPACE or ALTER TABLE ... IMPORT TABLESPACE statement.

##### end
This occurs at the end but before the cleanup of ALTER TABLE, CREATE VIEW, DELETE, INSERT, SELECT, or UPDATE statements.

##### executing
The thread has begun executing a statement.

##### Execution of init_command
The thread is executing statements in the value of the init_command system variable.

##### freeing items
The thread has executed a command. Some freeing of items done during this state involves the query cache. This state is usually followed by cleaning up.

##### Flushing tables
The thread is executing FLUSH TABLES and is waiting for all threads to close their tables.

##### FULLTEXT initialization
The server is preparing to perform a natural-language full-text search.

##### init
This occurs before the initialization of ALTER TABLE, DELETE, INSERT, SELECT, or UPDATE statements. Actions taken by the server in this state include flushing the binary log, the InnoDB log, and some query cache cleanup operations.  
For the end state, the following operations could be happening:  
	Removing query cache entries after data in a table is changed
	Writing an event to the binary log
	Freeing memory buffers, including for blobs

##### Killed
Someone has sent a KILL statement to the thread and it should abort next time it checks the kill flag. The flag is checked in each major loop in MySQL, but in some cases it might still take a short time for the thread to die. If the thread is locked by some other thread, the kill takes effect as soon as the other thread releases its lock.

##### logging slow query
The thread is writing a statement to the slow-query log.

##### NULL
This state is used for the SHOW PROCESSLIST state.

##### login
The initial state for a connection thread until the client has been authenticated successfully.

##### manage keys
The server is enabling or disabling a table index.

##### Opening tables, Opening table
The thread is trying to open a table. This is should be very fast procedure, unless something prevents opening. For example, an ALTER TABLE or a LOCK TABLE statement can prevent opening a table until the statement is finished. It is also worth checking that your table_open_cache value is large enough.

##### optimizing
The server is performing initial optimizations for a query.

##### preparing
This state occurs during query optimization.

##### preparing for alter table
The server is preparing to execute an in-place ALTER TABLE.

##### Purging old relay logs
The thread is removing unneeded relay log files.

##### query end
This state occurs after processing a query but before the freeing items state.

##### Reading from net
The server is reading a packet from the network.

##### Removing duplicates
The query was using SELECT DISTINCT in such a way that MySQL could not optimize away the distinct operation at an early stage. Because of this, MySQL requires an extra stage to remove all duplicated rows before sending the result to the client.

##### removing tmp table
The thread is removing an internal temporary table after processing a SELECT statement. This state is not used if no temporary table was created.

##### rename
The thread is renaming a table.

##### rename result table
The thread is processing an ALTER TABLE statement, has created the new table, and is renaming it to replace the original table.

##### Reopen tables
The thread got a lock for the table, but noticed after getting the lock that the underlying table structure changed. It has freed the lock, closed the table, and is trying to reopen it.

##### Repair by sorting
The repair code is using a sort to create indexes.

##### Repair done
The thread has completed a multi-threaded repair for a MyISAM table.

##### Repair with keycache
The repair code is using creating keys one by one through the key cache. This is much slower than Repair by sorting.

##### Rolling back
The thread is rolling back a transaction.

##### Saving state
For MyISAM table operations such as repair or analysis, the thread is saving the new table state to the .MYI file header. State includes information such as number of rows, the AUTO_INCREMENT counter, and key distributions.

##### Searching rows for update
The thread is doing a first phase to find all matching rows before updating them. This has to be done if the UPDATE is changing the index that is used to find the involved rows.

##### Sending data
The thread is reading and processing rows for a SELECT statement, and sending data to the client. Because operations occurring during this state tend to perform large amounts of disk access (reads), it is often the longest-running state over the lifetime of a given query.

##### setup
The thread is beginning an ALTER TABLE operation.

##### Sorting for group
The thread is doing a sort to satisfy a GROUP BY.

##### Sorting for order
The thread is doing a sort to satisfy a ORDER BY.

##### Sorting index
The thread is sorting index pages for more efficient access during a MyISAM table optimization operation.

##### Sorting result
For a SELECT statement, this is similar to Creating sort index, but for nontemporary tables.

##### statistics
The server is calculating statistics to develop a query execution plan. If a thread is in this state for a long time, the server is probably disk-bound performing other work.

##### System lock
The thread is going to request or is waiting for an internal or external system lock for the table. If this state is being caused by requests for external locks and you are not using multiple mysqld servers that are accessing the same MyISAM tables, you can disable external system locks with the --skip-external-locking option. However, external locking is disabled by default, so it is likely that this option will have no effect. For SHOW PROFILE, this state means the thread is requesting the lock (not waiting for it).

##### update
The thread is getting ready to start updating the table.

##### Updating
The thread is searching for rows to update and is updating them.

##### updating main table
The server is executing the first part of a multiple-table update. It is updating only the first table, and saving columns and offsets to be used for updating the other (reference) tables.

##### updating reference tables
The server is executing the second part of a multiple-table update and updating the matched rows from the other tables.

##### User lock
The thread is going to request or is waiting for an advisory lock requested with a GET_LOCK() call. For SHOW PROFILE, this state means the thread is requesting the lock (not waiting for it).

##### User sleep
The thread has invoked a SLEEP() call.

##### Waiting for commit lock
FLUSH TABLES WITH READ LOCK is waiting for a commit lock.

##### Waiting for global read lock
FLUSH TABLES WITH READ LOCK is waiting for a global read lock or the global read_only system variable is being set.

##### Waiting for tables, Waiting for table flush
The thread got a notification that the underlying structure for a table has changed and it needs to reopen the table to get the new structure. However, to reopen the table, it must wait until all other threads have closed the table in question.  
This notification takes place if another thread has used FLUSH TABLES or one of the following statements on the table in question: FLUSH TABLES tbl_name, ALTER TABLE, RENAME TABLE, REPAIR TABLE, ANALYZE TABLE, or OPTIMIZE TABLE.  

##### Waiting for lock_type lock
The server is waiting to acquire a lock, where lock_type indicates the type of lock:

	Waiting for event metadata lock

	Waiting for global read lock

	Waiting for schema metadata lock

	Waiting for stored function metadata lock

	Waiting for stored procedure metadata lock

	Waiting for table level lock

	Waiting for table metadata lock

	Waiting for trigger metadata lock

##### Waiting on cond
A generic state in which the thread is waiting for a condition to become true. No specific state information is available.

##### Writing to net
The server is writing a packet to the network.


Delayed-Insert Thread
---------------------
These thread states are associated with processing for DELAYED inserts (see Section 13.2.5.2, “INSERT DELAYED Syntax”). Some states are associated with connection threads that process INSERT DELAYED statements from clients. Other states are associated with delayed-insert handler threads that insert the rows. There is a delayed-insert handler thread for each table for which INSERT DELAYED statements are issued.  
States associated with a connection thread that processes an INSERT DELAYED statement from the client:  
##### allocating local table
The thread is preparing to feed rows to the delayed-insert handler thread.

##### Creating delayed handler
The thread is creating a handler for DELAYED inserts.

##### got handler lock
This occurs before the allocating local table state and after the waiting for handler lock state, when the connection thread gets access to the delayed-insert handler thread.

##### got old table
This occurs after the waiting for handler open state. The delayed-insert handler thread has signaled that it has ended its initialization phase, which includes opening the table for delayed inserts.

##### storing row into queue
The thread is adding a new row to the list of rows that the delayed-insert handler thread must insert.

##### waiting for delay_list
This occurs during the initialization phase when the thread is trying to find the delayed-insert handler thread for the table, and before attempting to gain access to the list of delayed-insert threads.

##### waiting for handler insert
An INSERT DELAYED handler has processed all pending inserts and is waiting for new ones.

##### waiting for handler lock
This occurs before the allocating local table state when the connection thread waits for access to the delayed-insert handler thread.

##### waiting for handler open
This occurs after the Creating delayed handler state and before the got old table state. The delayed-insert handler thread has just been started, and the connection thread is waiting for it to initialize.  
States associated with a delayed-insert handler thread that inserts the rows:  

##### insert
The state that occurs just before inserting rows into the table.

##### reschedule
After inserting a number of rows, the delayed-insert thread sleeps to let other threads do work.

##### upgrading lock
A delayed-insert handler is trying to get a lock for the table to insert rows.

##### Waiting for INSERT
A delayed-insert handler is waiting for a connection thread to add rows to the queue (see storing row into queue).


Query Cache Thread
------------------
These thread states are associated with the query cache (see Section 8.10.3, “The MySQL Query Cache”).  
##### checking privileges on cached query
The server is checking whether the user has privileges to access a cached query result.

##### checking query cache for query
The server is checking whether the current query is present in the query cache.

##### invalidating query cache entries
Query cache entries are being marked invalid because the underlying tables have changed.

##### sending cached result to client
The server is taking the result of a query from the query cache and sending it to the client.

##### storing result in query cache
The server is storing the result of a query in the query cache.

##### Waiting for query cache lock
This state occurs while a session is waiting to take the query cache lock. This can happen for any statement that needs to perform some query cache operation, such as an INSERT or DELETE that invalidates the query cache, a SELECT that looks for a cached entry, RESET QUERY CACHE, and so forth.


Replication Master Thread
-------------------------
The following list shows the most common states you may see in the State column for the master's Binlog Dump thread. If you see no Binlog Dump threads on a master server, this means that replication is not running—that is, that no slaves are currently connected.  
##### Sending binlog event to slave
Binary logs consist of events, where an event is usually an update plus some other information. The thread has read an event from the binary log and is now sending it to the slave.

##### Finished reading one binlog; switching to next binlog
The thread has finished reading a binary log file and is opening the next one to send to the slave.

##### Master has sent all binlog to slave; waiting for binlog to be updated
The thread has read all outstanding updates from the binary logs and sent them to the slave. The thread is now idle, waiting for new events to appear in the binary log resulting from new updates occurring on the master.

##### Waiting to finalize termination
A very brief state that occurs as the thread is stopping.


Replication Slave I/O Thread
----------------------------
The following list shows the most common states you see in the State column for a slave server I/O thread. This state also appears in the Slave_IO_State column displayed by SHOW SLAVE STATUS, so you can get a good view of what is happening by using that statement.  
##### Waiting for master update
The initial state before Connecting to master.

##### Connecting to master
The thread is attempting to connect to the master.

##### Checking master version
A state that occurs very briefly, after the connection to the master is established.

##### Registering slave on master
A state that occurs very briefly after the connection to the master is established.

##### Requesting binlog dump
A state that occurs very briefly, after the connection to the master is established. The thread sends to the master a request for the contents of its binary logs, starting from the requested binary log file name and position.

##### Waiting to reconnect after a failed binlog dump request
If the binary log dump request failed (due to disconnection), the thread goes into this state while it sleeps, then tries to reconnect periodically. The interval between retries can be specified using the CHANGE MASTER TO statement.

##### Reconnecting after a failed binlog dump request
The thread is trying to reconnect to the master.

##### Waiting for master to send event
The thread has connected to the master and is waiting for binary log events to arrive. This can last for a long time if the master is idle. If the wait lasts for slave_net_timeout seconds, a timeout occurs. At that point, the thread considers the connection to be broken and makes an attempt to reconnect.

##### Queueing master event to the relay log
The thread has read an event and is copying it to the relay log so that the SQL thread can process it.

##### Waiting to reconnect after a failed master event read
An error occurred while reading (due to disconnection). The thread is sleeping for the number of seconds set by the CHANGE MASTER TO statement (default 60) before attempting to reconnect.

##### Reconnecting after a failed master event read
The thread is trying to reconnect to the master. When connection is established again, the state becomes Waiting for master to send event.

##### Waiting for the slave SQL thread to free enough relay log space
You are using a nonzero relay_log_space_limit value, and the relay logs have grown large enough that their combined size exceeds this value. The I/O thread is waiting until the SQL thread frees enough space by processing relay log contents so that it can delete some relay log files.

##### Waiting for slave mutex on exit
A state that occurs briefly as the thread is stopping.


Replication Slave SQL Thread
----------------------------
The following list shows the most common states you may see in the State column for a slave server SQL thread:  
##### Waiting for the next event in relay log
The initial state before Reading event from the relay log.

##### Reading event from the relay log
The thread has read an event from the relay log so that the event can be processed.

##### Making temporary file (append) before replaying LOAD DATA INFILE
The thread is executing a LOAD DATA INFILE statement and is appending the data to a temporary file containing the data from which the slave will read rows.

##### Making temporary file (create) before replaying LOAD DATA INFILE
The thread is executing a LOAD DATA INFILE statement and is creating a temporary file containing the data from which the slave will read rows. This state can only be encountered if the original LOAD DATA INFILE statement was logged by a master running a version of MySQL earlier than version 5.0.3.

##### Slave has read all relay log; waiting for more updates
The thread has processed all events in the relay log files, and is now waiting for the I/O thread to write new events to the relay log.

##### Waiting for slave mutex on exit
A very brief state that occurs as the thread is stopping.

##### Waiting until MASTER_DELAY seconds after master executed event
The SQL thread has read an event but is waiting for the slave delay to lapse. This delay is set with the MASTER_DELAY option of CHANGE MASTER TO.

##### Killing slave
The thread is processing a STOP SLAVE statement.

##### Waiting for an event from Coordinator
Using the multi-threaded slave (slave_parallel_workers is greater than 1), one of the slave worker threads is waiting for an event from the coordinator thread.  

The Info column for the SQL thread may also show the text of a statement. This indicates that the thread has read an event from the relay log, extracted the statement from it, and may be executing it.  


Replication Slave Connection Thread
-----------------------------------
These thread states occur on a replication slave but are associated with connection threads, not with the I/O or SQL threads.  
##### Changing master
The thread is processing a CHANGE MASTER TO statement.

##### Killing slave
The thread is processing a STOP SLAVE statement.

##### Opening master dump table
This state occurs after Creating table from master dump.

##### Reading master dump table data
This state occurs after Opening master dump table.

##### Rebuilding the index on master dump table
This state occurs after Reading master dump table data.


MySQL Cluster Thread
--------------------
##### Committing events to binlog

##### Opening mysql.ndb_apply_status

##### Processing events

The thread is processing events for binary logging.

##### Processing events from schema table

The thread is doing the work of schema replication.

##### Shutting down

##### Syncing ndb table schema operation and binlog

This is used to have a correct binary log of schema operations for NDB.

##### Waiting for event from ndbcluster

The server is acting as an SQL node in a MySQL Cluster, and is connected to a cluster management node.

##### Waiting for first event from ndbcluster

##### Waiting for ndbcluster binlog update to reach current position

##### Waiting for ndbcluster to start

##### Waiting for schema epoch

The thread is waiting for a schema epoch (that is, a global checkpoint).

##### Waiting for allowed to take ndbcluster global schema lock

The thread is waiting for permission to take a global schema lock.

##### Waiting for ndbcluster global schema lock

The thread is waiting for a global schema lock held by another thread to be released.


Event Scheduler Thread
----------------------
These states occur for the Event Scheduler thread, threads that are created to execute scheduled events, or threads that terminate the scheduler.  
##### Clearing
The scheduler thread or a thread that was executing an event is terminating and is about to end.

##### Initialized
The scheduler thread or a thread that will execute an event has been initialized.

##### Waiting for next activation
The scheduler has a nonempty event queue but the next activation is in the future.

##### Waiting for scheduler to stop
The thread issued SET GLOBAL event_scheduler=OFF and is waiting for the scheduler to stop.

##### Waiting on empty queue
The scheduler's event queue is empty and it is sleeping.

